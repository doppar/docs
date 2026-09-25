---
title: Semantic Search with Embeds
description: Doppar #[Embeds] attribute documentation — local, attribute-driven semantic search for models
meta:
- name: keywords
  content: Embeds, Semantic Search, Vector, Embedding, pgvector, Doppar
---

## Embeds

### Introduction

Semantic search finds records by what they *mean*, not by which words they contain. A plain `WHERE description LIKE '%backpack%'` only matches the literal word — it will never find a product described as "rugged daypack for hiking," even though a shopper searching for "waterproof backpack" would consider that an excellent match.

Embeds adds this to Doppar models as a single attribute. Mark a column with `#[Embeds]`, and Doppar keeps a numeric representation of its meaning — a *vector* — in sync automatically.

Everything runs locally. There is no external API call, no API key to configure, and no per-request cost — embeddings are computed on your own server using a small local model.

## Installation

```bash
composer require doppar/embeds
```

Embeddings are computed via `doppar/ai`, which uses PHP's FFI extension to run the local model. FFI defaults to `ffi.enable = "preload"`, which permits ad-hoc CLI scripts but **blocks FFI inside any persistent server process** — PHP-FPM, or `php -S` (what `php pool server:start` uses). If embedding generation throws `Could not instantiate model for task: feature-extraction`, this is almost always why. Set, in the php.ini actually serving your requests:

```ini
ffi.enable=1
```

For PHP-FPM this can instead be set per-pool via `php_admin_value[ffi.enable] = true`, without touching the global php.ini. Either way, restart PHP-FPM afterward for it to take effect.

Register the launcher in `runtime/config/app.php`:

```php
"launchers" => [
    // ...
    \Doppar\Embeds\EmbedsLauncher::class,
],
```

Then publish and run migrations:

```bash
php pool vendor:publish --launcher="Doppar\Embeds\EmbedsLauncher"
```
Then run the migrations:

```bash
php pool migrate
```

## Defining an Embedded Property

Add the `Embeddable` trait to a model, then place `#[Embeds]` on the property whose meaning you want to search by.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Doppar\Embeds\Attributes\Embeds;
use Doppar\Embeds\Concerns\Embeddable;

class Product extends Model
{
    use Embeddable;

    protected $creatable = ['name', 'description'];

    #[Embeds]
    protected $description;
}
```

That's the entire contract. From this point on, saving a `Product` with a `description` keeps its embedding in sync automatically — nothing else in your application code changes.

## How Embeddings Are Computed

When `description` is set and the model is saved, Doppar sends the text through a local embedding model (`Xenova/all-MiniLM-L6-v2` by default) via `doppar/ai`, producing a 384-number vector that represents the text's meaning. That vector is stored, keyed to the model class, the record's id, and the attribute name — so multiple embedded columns on the same model are tracked independently.

```php
$product = new Product();
$product->name = 'Trail Pack 40L';
$product->description = 'rugged daypack for hiking in the rain';
$product->save();
```

the embedding is computed and stored right here when you call the `save()` method. You never call an embedding function yourself. Setting the property and saving is enough.

## Searching by Meaning

`whereSimilarTo()` is available on every model using `Embeddable`. Pass the embedded column, the text to search for, and how many results you want.

```php
$results = Product::whereSimilarTo(
    'description',
    'a durable waterproof backpack',
    limit: 10
);
```

`$results` is a `Collection` of `Product` models, already ordered from most to least similar. In this example, "rugged daypack for hiking in the rain" would rank highly even though it shares almost no words with the search phrase — the match is on meaning, not on spelling.

> **Note:** `whereSimilarTo()` returns a `Collection` directly, not a query builder. Ranking requires every candidate to be scored before results can be ordered, so unlike most query builder methods, it cannot stay lazily chainable — it is a terminal call, the same way `all()` and `get()` are.

## Choosing a Model per Property

`#[Embeds]` accepts an optional `model` argument if a property should use a different embedding model than the application default.

```php
#[Embeds(model: 'Xenova/paraphrase-multilingual-MiniLM-L12-v2')]
protected $description;
```

Leave it out to use the default (`Xenova/all-MiniLM-L6-v2`).

## Clearing an Embedded Value

Setting an embedded property to an empty string and saving removes its stored vector, rather than embedding an empty string.

```php
$product->description = '';
$product->save();
```

In this case when the `save()` method is called, the stored embedding for this record is deleted.

## How Embeds Fits Into save()

`#[Embeds]` is built entirely on top of Doppar's existing `#[Watches]` system — no framework changes were needed to wire it up. Every `#[Embeds]` property is registered as a watch behind the scenes, so the same lifecycle that already fires reactive property watchers is what fires embedding computation.

There is one lifecycle detail worth knowing: `#[Watches]` fires on the *update* path of `save()`, not on the very first `INSERT`. On its own, that would mean a model created with its embedded text already set would never get an embedding until its first update. Embeds accounts for this — a fresh record's embeddings are computed once, immediately after its first successful save — so the example under [Defining an Embedded Property](#defining-an-embedded-property) works exactly as shown, with no follow-up update required.

## Vector Index Drivers

Storing and ranking vectors is handled by a driver, resolved from `runtime/config/embeds.php`:

```php
'driver' => env('EMBEDS_DRIVER', 'brute_force'),
```

Both drivers sit behind the exact same API — `#[Embeds]` and `whereSimilarTo()` never change. Switching drivers is a configuration change, not a code change.

### The Brute Force Driver

The default. Vectors are stored as JSON in a plain `embeddings` table, and `whereSimilarTo()` ranks every stored vector for that column in PHP at search time.

- Works on every database Doppar supports, SQLite included.
- Zero setup — no extensions, no additional configuration.
- Fine for small-to-medium tables (a few thousand rows). It does not scale to millions of rows, because every search scans every stored vector for the column.

### The pgvector Driver

A real vector index, backed by PostgreSQL's [pgvector](https://github.com/pgvector/pgvector) extension. Ranking happens **inside the database**, using an HNSW index — PHP never loads or scores candidate vectors itself.

Requires:

- A PostgreSQL connection.
- The `pgvector` extension installed on that server (`CREATE EXTENSION vector`).

To enable it:

```
EMBEDS_DRIVER=pgvector
EMBEDS_PGVECTOR_CONNECTION=pgsql
```

Then run migrations again — the `pgvector`-only migration creates its own `embedding_vectors` table with a native `vector` column and an HNSW index, and is skipped automatically unless `EMBEDS_DRIVER` is set to `pgvector`.

> **Note:** MySQL and SQLite are intentionally not offered as indexed drivers. SQLite has no practical native vector index. MySQL only gained native vector search in version 9.0 (released 2024), which is not yet what most production MySQL installs are running — a MySQL driver may be added once that changes. Both remain fully supported through the brute force driver.

### The Redis Driver

A real vector index backed by [Redis Stack](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/vector-search/) (the RediSearch module). Same guarantee as pgvector — ranking happens inside Redis via an HNSW index, not in PHP — which makes it a good fit if Redis is already part of your stack rather than PostgreSQL.

Requires:

- A Redis server with the RediSearch module loaded (Redis Stack, or Redis Enterprise). Plain/community Redis does not include this — check with `redis-cli MODULE LIST`; an empty result means it's not loaded.
- `predis/predis`, which this package does not install for you:

```bash
composer require predis/predis
```

#### Getting RediSearch

RediSearch is a loadable module (a `.so` file), not a separate database — if you already have Redis running, you do **not** need a second server or a different port, *provided your Redis core is version 7.1.0 or newer*. RediSearch 7.x hard-requires it and refuses to load otherwise, logging `Redis version is too old, please upgrade to redis 7.1.0 and above` and **aborting the server it was asked to load into**. Check first:

Make sure your `redis stack server` module list shows the `search`
```bash
redis-cli -p 6380 MODULE LIST | grep -i search
```
To enable it:

```
EMBEDS_DRIVER = "redis"
EMBEDS_REDIS_HOST = "127.0.0.1"

# connected port of redis stack server
EMBEDS_REDIS_PORT = 6379
```

Unlike the other two drivers, this one needs no migration — there is no SQL schema to create. The RediSearch index is created automatically, the first time a vector is indexed or searched.

### Choosing Between Them

Start with the brute force driver — it requires no setup and is correct for anything up to a few thousand embedded records. Move to `pgvector` or `redis` once either becomes true: the table is large enough that scan time is noticeable, or you're already running one of those two systems and enabling vector search on it costs you nothing. Between the two, let your existing infrastructure decide — pick `pgvector` if PostgreSQL is your primary database, `redis` if Redis is already load-bearing in your stack.

## Limitations

- **No automatic re-embedding.** If you change the embedding model (via the `model` argument or the app default), previously stored vectors are not recomputed. Vectors from different models are not comparable to each other — clear and repopulate the affected column's embeddings after switching models.
- **The brute force driver does not scale indefinitely.** It is correct, not fast, at large row counts. Move to `pgvector` for tables where that matters.
- **`pgvector` requires the extension to be installed on the server.** Doppar cannot install a PostgreSQL extension for you — that's a one-time `CREATE EXTENSION vector` your database administrator or hosting provider needs to run.