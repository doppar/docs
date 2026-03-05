---
title: AI
description: Doppar AI component for transformers and LLM agents
meta:
  - name: keywords
    content: ai transformers llm agents machine learning
---

## AI
### Introduction
Doppar AI is a powerful component that brings advanced artificial intelligence capabilities to your PHP applications. It is built on top of **[Symfony AI](https://github.com/symfony/ai)** and **[Transformers.php](https://github.com/CodeWithKyrian/transformers-php)**, providing a smooth integration of machine learning models and large language models (LLMs) into your Doppar ecosystem.

It provides two main components: **`Pipeline`** for running transformer-based machine learning tasks locally, and **`Agent`** for interacting with cloud-based large language models (LLMs) like OpenAI and Gemini. Whether you need sentiment analysis, text generation, image classification, or conversational AI, Doppar AI makes it simple and accessible.

The component leverages the Transformers.php library to run machine learning models directly on your server, eliminating the need for external API calls for many tasks. For advanced conversational AI and complex reasoning, the Agent component provides a fluent interface to interact with state-of-the-art language models from OpenAI, Google Gemini, or your own self-hosted models.

 - [Features](#features)
 - [Installation](#installation)
 - [Quick Start](#quick-start)
 - [Pipeline Tasks](#pipeline-tasks)
 - [Agent Usage](#agent-usage)
 - [Streaming Responses](#streaming-responses)
 - [Agent Persistence](#agent-persistence)
 - [Advanced Usage](#advanced-usage)
 - [Rate Limiting for Agents](#rate-limiting-for-agents)

 ## Features
Doppar AI is designed to be versatile, easy to use, and powerful. It brings modern AI capabilities directly to your PHP applications without the complexity of traditional machine learning implementations.

- **15+ Transformer Tasks** - Sentiment analysis, text generation, translation, QA, and more
- **Multiple LLM Support** - OpenAI, Google Gemini, Anthropic Claude, OpenRouter and self-hosted models
- **Local Model Execution** - Run models on your server without external API calls
- **Fluent Agent API** - Build conversational AI with ease
- **Image Processing** - Classification, object detection, captioning
- **Zero-Shot Learning** - Classify without training data
- **Quantized Models** - Optimize performance and memory usage
- **Custom Model Support** - Use any HuggingFace model
- **Query Helper** - Ask questions about structured data
- **Framework Integration** - Seamless integration with Doppar framework

## Installation
You may install Doppar AI via the composer require command:
```bash
composer require doppar/ai
```

### Register Provider
Next, register the AI service provider so that Doppar can initialize it properly. Open your `config/app.php` file and add the `AIServiceProvider` to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\AI\AIServiceProvider::class,
],
```

### Verify Installation
Make sure `FFI` is enabled in your `php.ini` file. Do not let it in preload mode. OR : start php server with `-d ffi.enable=1`
```bash
php -d ffi.enable=1 -S localhost:8000 -t public server.php 
```

You can verify the installation now by running the AI command:
```bash
php pool ai:run "Hello, how are you ?"
```

This will use a small text generation model to respond to your prompt. On first run, the model will be downloaded and cached in your storage directory.

## Quick Start
Doppar AI provides two main ways to work with AI: **`Pipeline`** for transformer tasks and **`Agent`** for LLM interactions. Let's start with simple examples of each

### Pipeline for Sentiment Analysis
Here you can perform sentiment analysis using the Pipeline component. The pipeline processes text and determines whether the sentiment is positive, negative, or neutral.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::SENTIMENT_ANALYSIS,
    data: 'I absolutely love this product! Best purchase ever!'
);

// Output: [['label' => 'POSITIVE', 'score' => 0.9998]]
```

### Agent for Conversational AI
This section demonstrates how to use the Agent component to communicate with cloud-based large language models such as OpenAI's GPT series. It allows your application to generate responses, explanations, and conversational outputs with zero configuration.

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-3.5-turbo')
    ->prompt('Explain quantum computing in simple terms')
    ->send();

echo $response; // Returns the AI-generated explanation
```

## Pipeline Tasks
The Pipeline component supports 15+ different transformer tasks. Each task can run locally on your server using pre-trained models from [HuggingFace](https://huggingface.co/models). You can use any `HuggingFace` model as per your business logic.

### Available Tasks
| Task                              | Description                                          | Use Case                          |
| --------------------------------- | ---------------------------------------------------- | --------------------------------- |
| `SENTIMENT_ANALYSIS`              | Analyze emotional tone of text                       | Reviews, feedback analysis        |
| `TEXT_GENERATION`                 | Generate new text from prompts                       | Content creation, chatbots        |
| `TEXT_CLASSIFICATION`             | Categorize text into predefined labels               | Topic classification, spam filter |
| `TOKEN_CLASSIFICATION`            | Token-level classification (NER, POS)                | Entity extraction, tagging        |
| `QUESTION_ANSWERING`              | Answer questions based on context                    | Knowledge bases, search           |
| `TRANSLATION`                     | Translate text between languages                     | Multilingual apps                 |
| `SUMMARIZATION`                   | Create concise summaries                             | Article summarization             |
| `FILL_MASK`                       | Predict masked words                                 | Auto-complete, suggestions        |
| `ZERO_SHOT_CLASSIFICATION`        | Classify without training data                       | Dynamic categorization            |
| `FEATURE_EXTRACTION`              | Extract numerical features                           | Semantic search, clustering       |
| `EMBEDDING`                       | Generate text embeddings                             | Similarity matching, search       |
| `IMAGE_CLASSIFICATION`            | Classify images into categories                      | Photo organization                |
| `IMAGE_CAPTION`                   | Generate image descriptions                          | Accessibility, content discovery  |
| `ZERO_SHOT_IMAGE_CLASSIFICATION`  | Classify images without training                     | Flexible image categorization     |
| `OBJECT_DETECTION`                | Detect and locate objects in images                  | Security, inventory               |

### Sentiment Analysis
Analyze the emotional tone of text to determine if it's positive, negative, or neutral.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

// Basic sentiment analysis
$result = Pipeline::execute(
    task: TaskEnum::SENTIMENT_ANALYSIS,
    data: 'This movie was terrible and boring.'
);

// Output: [['label' => 'NEGATIVE', 'score' => 0.9995]]
```

### With Custom HuggingFace Model
You can pass the custom `HuggingFace` model as you want like this way
```php
$result = Pipeline::execute(
    task: TaskEnum::SENTIMENT_ANALYSIS,
    data: 'I love doppar',
    model: 'Xenova/distilbert-base-uncased-finetuned-sst-2-english',
    quantized: true
);
```

### Text Generation
Generate new text based on prompts or continue existing text. Perfect for chatbots and content creation.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$messages = [
    ['role' => 'user', 'content' => 'Write a haiku about PHP programming']
];

$result = Pipeline::execute(
    task: TaskEnum::TEXT_GENERATION,
    messages: $messages,
    maxNewTokens: 100
);

echo $result[0]['generated_text'];
```

With Custom Model

```php
$messages = [
    ['role' => 'system', 'content' => 'You are a helpful coding assistant.'],
    ['role' => 'user', 'content' => 'Explain what is dependency injection']
];

$result = Pipeline::execute(
    task: TaskEnum::TEXT_GENERATION,
    model: 'HuggingFaceTB/SmolLM2-360M-Instruct',
    messages: $messages,
    maxNewTokens: 256,
    returnFullText: false
);
```

### Translation
This example shows how you can easily translate the same text into different languages by simply changing the target language parameter. The Pipeline component makes multilingual translation straightforward and flexible. Translate text from one language to another.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::TRANSLATION,
    data: 'Hello, how are you?',
    tgtLang: 'fr', // French
    maxNewTokens: 100
);

// Output: 'Bonjour, comment allez-vous?'
```

### Localization & AI-powered translation
Doppar AI also provides native translation and localization support built around your `/lang/{lang}` folders. You can translate arbitrary content between languages with your preferred AI agent, or automatically generate a brand new locale by translating an entire Doppar translation directory.

Translate any content between two languages using an Agent:

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$content = 'Welcome to Doppar!';

$translated = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-3.5-turbo')
    ->translate('en', 'fr', $content);

// e.g. "Bienvenue sur Doppar !"
```

Automatically translate your Doppar translation folder (`/lang/{lang}`) and create a new locale from a controller or service:

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\Gemini;

// This will read all files from /lang/fr and create a new /lang/br folder
$files = Agent::using(Gemini::class)
    ->withKey(env('GEMINI_API_KEY'))
    ->model('gemini-2.0-flash')
    ->translateLocalization('fr', 'br');

// $files now contains the list of translated files
```

You can also trigger localization from the CLI using the `ai:translate` command:

```bash
php pool ai:translate gemini fr br

# This will ask you for your API key
# ai:translate {agent} {langFrom} {langTo} {model?}
# Currently, gemini, claude and openai agents are supported
```

### Question Answering
Pipeline component can extract precise answers from a provided block of text. By supplying both a context and a question, the model identifies the most relevant answer based on the information available. Extract answers from a given context based on questions.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$context = "Doppar is a modern PHP framework created for building robust web applications. Mahedi Hasan is the creator of doppar.";

$result = Pipeline::execute(
    task: TaskEnum::QUESTION_ANSWERING,
    question: 'Who is the creator of doppar?',
    context: $context,
    topK: 1
);

// Output: ['answer' => 'Mahedi Hasan', 'score' => 0.95]
```

### Zero-Shot Classification
Classify text into categories without any training data or examples.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::ZERO_SHOT_CLASSIFICATION,
    data: 'The weather is beautiful today with clear blue skies.',
    candidateLabels: ['weather', 'sports', 'politics', 'technology']
);

// Output:
// [
//     'labels' => ['weather', 'technology', 'sports', 'politics'],
//     'scores' => [0.95, 0.03, 0.01, 0.01]
// ]
```

Product categorization example:
```php
$product = "Latest iPhone with 5G connectivity and amazing camera";

$result = Pipeline::execute(
    task: TaskEnum::ZERO_SHOT_CLASSIFICATION,
    data: $product,
    candidateLabels: ['electronics', 'clothing', 'food', 'books']
);
```

### Text Classification
By specifying a suitable model and providing text input, the system returns the most likely label along with a confidence score. Categorize text into predefined classes like this way.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::TEXT_CLASSIFICATION,
    data: 'I am very happy with this service!',
    model: 'Xenova/distilbert-base-uncased-finetuned-sst-2-english'
);

// Output: [['label' => 'POSITIVE', 'score' => 0.9998]]
```


### Summarization
Generate concise summaries of long text.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$text = "Long article text here... (can be multiple paragraphs)";

$result = Pipeline::execute(
    task: TaskEnum::SUMMARIZATION,
    data: $text,
    maxNewTokens: 150
);

echo $result[0]['summary_text'];
```

### Fill Mask
Predict masked words in sentences. Useful for autocomplete and suggestions.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::FILL_MASK,
    data: 'The capital of France is [MASK].',
    topK: 3
);

// Output:
// [
//     ['token_str' => 'Paris', 'score' => 0.98],
//     ['token_str' => 'Lyon', 'score' => 0.01],
//     ['token_str' => 'Marseille', 'score' => 0.005]
// ]
```

### Token Classification
The Pipeline component identifies entities within a sentence and labels each token with its corresponding entity type using `TOKEN_CLASSIFICATION`. Perform token-level classification like Named Entity Recognition (NER).
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::TOKEN_CLASSIFICATION,
    data: 'My name is John and I work at Google in California.'
);

// Output:
// [
//     ['entity' => 'B-PER', 'word' => 'John', 'score' => 0.99],
//     ['entity' => 'B-ORG', 'word' => 'Google', 'score' => 0.98],
//     ['entity' => 'B-LOC', 'word' => 'California', 'score' => 0.97]
// ]
```

### Feature Extraction
Extract numerical feature vectors from text for semantic analysis.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::FEATURE_EXTRACTION,
    data: 'Machine learning is fascinating'
);

// Output: Multi-dimensional array of numerical features
```

### Image Classification
Use the Pipeline component to classify images into predefined categories. Provide the image path or URL, and the model will return the top predicted labels along with their confidence scores. Classify images into predefined categories.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::IMAGE_CLASSIFICATION,
    imageUrl: '/path/to/image.jpg',
    topK: 3
);

// Output:
// [
//     ['label' => 'golden retriever', 'score' => 0.95],
//     ['label' => 'labrador', 'score' => 0.03],
//     ['label' => 'dog', 'score' => 0.02]
// ]
```


### Image Caption
Generate descriptive captions for images.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::IMAGE_CAPTION,
    imageUrl: '/path/to/photo.jpg',
    maxNewTokens: 50
);

// Output: "a dog playing in the park on a sunny day"
```

### Zero-Shot Image Classification
Perform image classification without requiring any training data. By providing candidate labels, the model predicts which labels best match the content of the image, along with confidence scores. Let's see the example of classifying images without training data.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::ZERO_SHOT_IMAGE_CLASSIFICATION,
    imageUrl: '/path/to/image.jpg',
    candidateLabels: ['cat', 'dog', 'bird', 'fish']
);

// Output:
// [
//     'labels' => ['dog', 'cat', 'bird', 'fish'],
//     'scores' => [0.92, 0.05, 0.02, 0.01]
// ]
```

### Object Detection
Use the Pipeline component to detect and locate objects within images. The model returns detected objects, their confidence scores, and bounding box coordinates for precise localization. Detect and locate objects within images like this way.
```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::OBJECT_DETECTION,
    imageUrl: '/path/to/image.jpg',
    threshold: 0.5
);

// Output:
// [
//     [
//         'label' => 'person',
//         'score' => 0.98,
//         'box' => ['xmin' => 100, 'ymin' => 50, 'xmax' => 300, 'ymax' => 400]
//     ],
//     [
//         'label' => 'car',
//         'score' => 0.85,
//         'box' => ['xmin' => 400, 'ymin' => 200, 'xmax' => 700, 'ymax' => 500]
//     ]
// ]
```

### Higher Confidence Threshold
Increase the detection threshold to only return objects with higher confidence scores. This ensures that only the most likely predictions are returned, reducing false positives.
```php
$result = Pipeline::execute(
    task: TaskEnum::OBJECT_DETECTION,
    imageUrl: '/path/to/image.jpg',
    threshold: 0.8, // Only return high-confidence detections
    model: 'Xenova/detr-resnet-50'
);
```

### Automatic Speech Recognition (ASR)
Use Automatic Speech Recognition to convert spoken audio into text. This task processes an audio file and returns the transcribed speech as plain text.

```php
$output = Pipeline::execute(
    task: TaskEnum::AUTOMATIC_SPEECH_RECOGNITION,
    audioPath: public_path('assets/speech-94649.mp3')
);

dd($output);
```

Output
```php
["text" => "You are just a line of code."]
```
This task is useful for transcribing voice recordings, interviews, podcasts, or any audio content where extracting text is required. The returned text value contains the model’s best transcription of the provided audio

## Agent Usage
The Agent component provides a fluent interface for interacting with large language models. It supports OpenAI, Google Gemini, Claude, OpenRouter and self-hosted models.

### Supported Agents
| Agent        | Class                                  | Requirements                        |
| ------------ | -------------------------------------- | ----------------------------------- |
| OpenAI       | `Doppar\AI\AgentFactory\Agent\OpenAI`        | OpenAI API key                |
| Google Gemini| `Doppar\AI\AgentFactory\Agent\Gemini`        | Google AI API key             |
| Claude Anthropic| `Doppar\AI\AgentFactory\Agent\Claude`     | Claude Anthropic AI API key   |
| OpenRouter| `Doppar\AI\AgentFactory\Agent\OpenRouter`     | OpenRouter AI API key   |
| Self-hosted  | `Doppar\AI\AgentFactory\Agent\SelfHost`      | LM Studio or compatible host  |

### Quick Start with OpenAI
This example shows how to quickly set up an Agent to interact with OpenAI's GPT models. By providing your API key, selecting a model, and sending a prompt, you can generate intelligent responses for conversational AI, explanations, or content generation with minimal setup.
```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-3.5-turbo')
    ->prompt('Explain Doppar middleware in 3 sentences')
    ->send();

echo $response;
```

You can also use `make()` method to send your promt like this way
```php
$response = Agent::make(OpenAI::class, env('OPEN_AI_API_KEY'))
    ->model('gpt-3.5-turbo')
    ->prompt('Hello, how are you?')
    ->maxTokens(100)
    ->send();
```

### Using Google Gemini

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\Gemini;

$response = Agent::using(Gemini::class)
    ->withKey(env('GEMINI_API_KEY'))
    ->model('gemini-2.0-flash')
    ->prompt('What are the SOLID principles?')
    ->temperature(0.7)
    ->maxTokens(500)
    ->send();

echo $response;
```
### Using Claude Anthropic

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\Claude;

$response = Agent::using(Claude::class)
    ->withKey(env('CLAUDE_API_KEY'))
    ->model('claude-sonnet-4-5-20250929')
    ->prompt('Hello this is my prompt!')
    ->temperature(0.7)
    ->maxTokens(500)
    ->send();
```
### Using OpenRouter

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenRouter;

$response = Agent::using(OpenRouter::class)
    ->withKey(env('OPENROUTER_API_KEY'))
    ->model('openrouter/free')
    ->prompt('I am using OpenRouter!')
    ->temperature(0.7)
    ->maxTokens(500)
    ->send();
```

### Self-Hosted Models
This section shows how to run your own AI models locally using LM Studio or any compatible platform. By specifying the host URL and optional key, you can interact with your self-hosted model, send prompts, and receive responses without relying on external APIs.

Run your own models with LM Studio or compatible platforms.
```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\SelfHost;

$response = Agent::using(SelfHost::class)
    ->withHost('http://localhost:1234')
    ->withKey('optional-key') // Optional for local models
    ->model('local-model-name')
    ->prompt('Generate a PHP function to validate email')
    ->send();
```

### System Messages
System messages allow you to define the context and behavior of the AI, guiding it to respond in a specific tone, role, or style.
```php
$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->system('You are a senior PHP developer who writes clean, modern code.')
    ->prompt('Write a repository pattern example')
    ->send();
```

### Multiple Messages
For more complex conversations, you can send multiple messages to the AI, including system, user, and assistant roles. This helps maintain context across turns and enables richer, interactive dialogues.
```php
$messages = [
    ['role' => 'system', 'content' => 'You are a helpful coding assistant.'],
    ['role' => 'user', 'content' => 'What is dependency injection?'],
];

$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-3.5-turbo')
    ->messages($messages)
    ->send();
```

### Fluent Message Building
Fluent message building allows you to interact with the AI in a step-by-step manner, adding system instructions, prompts, and additional messages in a readable, chainable syntax. This approach makes it easy to construct complex queries or conversational flows.
```php
$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->system('You are a database expert.')
    ->prompt('Explain database indexing')
    ->message(['role' => 'user', 'content' => 'Give me an example with MySQL'])
    ->send();
```

### Customizing Parameters
You can control the AI's behavior, creativity, and output length by adjusting parameters such as temperature and maxTokens. Higher temperature values produce more creative responses, while lower values make output more deterministic.
```php
$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->temperature(0.9) // Higher = more creative
    ->maxTokens(1000)  // Maximum response length
    ->prompt('Write a creative story about AI')
    ->send();
```

Conservative Settings (for factual responses)
```php
$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->temperature(0.2) // Lower = more deterministic
    ->maxTokens(300)
    ->prompt('What is the capital of France?')
    ->send();
```

### Advanced Parameters
For finer control, you can pass additional parameters directly to the underlying API, including `top_p`, `presence_penalty`, and `frequency_penalty`. Pass additional parameters to the underlying API like this way.
```php
$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->withParams([
        'temperature' => 0.8,
        'top_p' => 0.9,
        'presence_penalty' => 0.6,
        'frequency_penalty' => 0.5
    ])
    ->prompt('Generate unique product descriptions')
    ->send();
```

### Getting Complete Response
Get the full response object instead of just text.
```php
$fullResponse = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->prompt('Explain APIs')
    ->complete()
    ->send();

// Access detailed information
dd($fullResponse);
```

### Create reusable agent instances
You can create reusable Agent instances for efficient and consistent interactions with the same model configuration. This is useful when you need to make multiple queries without re-initializing the agent each time.
```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$agent = Agent::make(OpenAI::class, env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->temperature(0.7);

// Use multiple times
$response1 = $agent->prompt('First question')->send();
$response2 = $agent->prompt('Second question')->send();
```

## Streaming Responses
For real-time, token-by-token output from large language models, Doppar AI supports streaming responses. Instead of waiting for the full response to be generated, streaming lets you display content progressively as it arrives — ideal for chat interfaces, long-form generation, and interactive applications.

Streaming is supported for all cloud-based agents: `OpenAI`, `Gemini`, `Claude`, `OpenRouter`, and `SelfHost`.

- Streaming with `withStreaming()`
- Streaming with `stream()`
- Streaming with System Messages
- Streaming with Parameters

### Streaming with `withStreaming()`
The `withStreaming()` method enables streaming mode on the agent before calling `send()`. The returned value is a `Generator` that you iterate over to receive each chunk as it arrives.
```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$stream = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->prompt('Write a short story about a robot learning to paint.')
    ->withStreaming()
    ->send();

foreach ($stream as $chunk) {
    echo $chunk;
    flush();
}
```

### Streaming with `stream()`
Alternatively, you can call `stream()` directly instead of `withStreaming()->send()`. Both approaches are equivalent and return a `Generator`.
```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenRouter;

$stream = Agent::using(OpenRouter::class)
    ->withKey(env('OPENROUTER_API_KEY'))
    ->model('openrouter/free')
    ->prompt('What is the capital of France?')
    ->stream();

foreach ($stream as $chunk) {
    echo $chunk;
    flush();
}
```

### Streaming with System Messages
You can combine streaming with system instructions, multi-turn messages, and all other fluent methods just as you would in a standard non-streaming request.
```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\Claude;

$stream = Agent::using(Claude::class)
    ->withKey(env('CLAUDE_API_KEY'))
    ->model('claude-sonnet-4-5-20250929')
    ->system('You are a concise technical writer. Always use code examples.')
    ->prompt('Explain the difference between interfaces and abstract classes in PHP.')
    ->stream();

foreach ($stream as $chunk) {
    echo $chunk;
    flush();
}
```

### Streaming with Parameters
Temperature, max tokens, and any other parameters work seamlessly with streaming.
```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\Gemini;

$stream = Agent::using(Gemini::class)
    ->withKey(env('GEMINI_API_KEY'))
    ->model('gemini-2.0-flash')
    ->temperature(0.8)
    ->maxTokens(800)
    ->prompt('Generate a detailed marketing plan for a SaaS product.')
    ->withStreaming()
    ->send();

foreach ($stream as $chunk) {
    echo $chunk;
    flush();
}
```

> **Note** When streaming, the response is a Generator and cannot be used as a plain string. Do not pass a streamed response to `store()` directly — resolve the full text first by concatenating chunks, then store it.

## Agent Persistence
Doppar AI includes a message persistence system that allows you to save and restore conversation history across requests. This enables stateful, multi-turn conversations where context is preserved between separate HTTP requests or CLI executions.

The persistence system is built around two components: a `StoreInterface` contract for defining custom storage backends, and a `CacheStore` implementation as the first ready-to-use integration.

### StoreInterface
`StoreInterface` defines the contract that all storage backends must implement. You can create your own implementation backed by a `database`, `Redis`, `sessions`, or any other storage mechanism.
```php
namespace Doppar\AI\Store;

interface StoreInterface
{
    public function store(string $key, mixed $data): bool;
    public function load(string $key): mixed;
    public function has(string $key): bool;
    public function delete(string $key): bool;
    public function clear(): bool;
}
```

| **Method**         | **Description**                             |
| ------------------ | ------------------------------------------- |
| `store(key, data)` | Persist message history under the given key |
| `load(key)`        | Retrieve message history by key             |
| `has(key)`         | Check whether a key exists in the store     |
| `delete(key)`      | Remove a specific key from the store        |
| `clear()`          | Wipe all entries from the store             |

### CacheStore
`CacheStore` is the built-in file-based implementation of `StoreInterface`. It serializes message arrays to disk using PHP's native serialization, with MD5-based filenames to avoid collisions. It is intended as a reference implementation — in production, you will typically replace this with a database or Redis-backed store suited to your application.
```php
use Doppar\AI\Store\CacheStore;

// Default path: ./doppar_ai_store
$store = new CacheStore();

// Custom path
$store = new CacheStore('./storage/doppar_ai_store');
```
The store directory is created automatically if it does not exist.

#### Saving Conversations
Use `withStore()` to attach a store instance to your agent, then call `store()` after receiving a response to persist the full conversation history including the assistant's reply.
```php
use Doppar\AI\Agent;
use Doppar\AI\Store\CacheStore;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$store = new CacheStore('./storage/doppar_ai_store');

$agent = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->withStore($store)
    ->system('You are a helpful assistant.')
    ->prompt('What is dependency injection?');

$response = $agent->send();

// Persist the conversation including the assistant response
$agent->store('conversation-user-42', $response);

echo $response;
```
When `store()` is called with a response string, it automatically appends the assistant's message to the history before saving, so the full exchange is preserved.

#### Loading and Continuing Conversations
Use `loadMessages()` to restore a previously saved conversation, then continue from where it left off.
```php
use Doppar\AI\Agent;
use Doppar\AI\Store\CacheStore;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$store = new CacheStore('./storage/doppar_ai_store');

$agent = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->withStore($store);

// Restore full history and continue the conversation
$response = $agent->loadMessages('conversation-user-42')
    ->prompt('Can you give me a concrete PHP example of that?')
    ->send();

// Save the updated history
$agent->store('conversation-user-42', $response);

echo $response;
```
The model receives the full prior context and responds as if the conversation has been continuous.

#### Full Multi-Turn Example
This example shows a complete stateful conversation across three separate turns, as might happen across multiple HTTP requests in a web application.

```php
use Doppar\AI\Agent;
use Doppar\AI\Store\CacheStore;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$store = new CacheStore('./storage/doppar_ai_store');
$conversationKey = 'chat-' . auth()->id();

$agent = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->withStore($store)
    ->system('You are a senior PHP developer who gives concise, practical advice.');

// Turn 1 — first request
$response = $agent->prompt('What is the repository pattern?')->send();
$agent->store($conversationKey, $response);
echo $response;

// Turn 2 — second request (e.g. next HTTP request)
$response = $agent->loadMessages($conversationKey)
    ->prompt('Show me a simple implementation in PHP.')
    ->send();
$agent->store($conversationKey, $response);
echo $response;

// Turn 3 — third request
$response = $agent->loadMessages($conversationKey)
    ->prompt('How would I test this with PHPUnit?')
    ->send();
$agent->store($conversationKey, $response);

echo $response;
```

### Custom Store Implementations
`CacheStore` is a starting point. In production you will want persistence backed by your database, Redis, or user sessions. Implement `StoreInterface` to plug in any backend.

Database-backed example:
```php
use Doppar\AI\Store\StoreInterface;

class DatabaseStore implements StoreInterface
{
    public function store(string $key, mixed $data): bool
    {
        // store
    }

    public function load(string $key): mixed
    {
        // load
    }

    public function has(string $key): bool
    {
        // has
    }

    public function delete(string $key): bool
    {
        // delete
    }

    public function clear(): bool
    {
        // clear
    }
}
```

Then use it exactly the same way as `CacheStore:`
```php
$store = new DatabaseStore();

$agent = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->withStore($store)
    ->loadMessages('conversation-user-42')
    ->prompt('Continue our discussion about SOLID principles.')
    ->send();

$agent->store('conversation-user-42', $response);
```

Use a consistent, user-scoped key such as `'chat-' . auth()->id()` to isolate conversations per user and avoid history collisions in multi-user applications.

## Advanced Usage
### Query Helper
The Query Helper allows you to interact with structured data, such as arrays, objects, or strings, and ask questions about their content. This provides a simple way to extract insights or verify information without manually parsing the data.
```php
use Doppar\AI\Pipeline;

$user = [
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'age' => 28,
    'city' => 'New York'
];

$result = Pipeline::query(
    item: $user,
    question: 'Is this user over 25 years old?'
);

// Returns: true or false
```

With custom model:
```php
$product = [
    'name' => 'Laptop',
    'price' => 1200,
    'category' => 'Electronics',
    'in_stock' => true
];

$result = Pipeline::query(
    item: $product,
    question: 'Is this product expensive?',
    model: 'custom-qa-model',
    topK: 1
);
```

With query objects:
```php
$order = $orderRepository->find(123);

$isUrgent = Pipeline::query(
    item: $order,
    question: 'Is this order urgent?'
);

if ($isUrgent) {
    // Handle urgent order
}
```

Query strings:
```php
$feedback = "The product quality is excellent but delivery was slow.";

$isPositive = Pipeline::query(
    item: $feedback,
    question: 'Is this feedback positive overall?'
);
```

### Vector Helper for RAG (Retrieval Augmented Generation)
The `Doppar\AI\Vector\Vector` helper provides utility methods to work with text embeddings and build Retrieval Augmented Generation (RAG) workflows on top of Doppar AI.

It is designed to:
- Compute similarity between vectors using cosine similarity.
- Select the most relevant context chunks for a question.
- Generate embeddings from an `Agent` (currently supported with `OpenAI` only).

#### cosineSimilarity
Compute the cosine similarity between two numeric vectors. This is used internally to compare a question embedding with your document embeddings.

```php
use Doppar\AI\Vector\Vector;

$similarity = Vector::cosineSimilarity($vectorA, $vectorB); // float between -1 and 1
```

#### getContext
Given a list of precomputed document vectors and a question vector, `getContext` sorts all chunks by similarity and returns a concatenated text of the top matches.

Expected structure of `$docs`:

```php
[
    [
        'vector'  => [/* array of floats */],
        'content' => 'Markdown or text content here...',
    ],
    // ... more chunks
]
```

Usage:

```php
use Doppar\AI\Vector\Vector;

$context = Vector::getContext($docs, $questionVector);
```

#### embedding
Generate an embedding vector for a given text using a configured `Agent`. Embeddings are currently supported only for the `OpenAI` agent.

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;
use Doppar\AI\Vector\Vector;

$agent = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'));

$vector = Vector::embedding($agent, 'text-embedding-3-small', 'Some text to embed');
```

#### End-to-End RAG Example with OpenAI
The following example shows how to:
- Load precomputed document embeddings from a local JSON cache (created beforehand, e.g. with `Vector::embedding` on each markdown file).
- Embed a user question with `Vector::embedding`.
- Retrieve the most relevant context with `Vector::getContext`.
- Call a Doppar `Agent` (OpenAI) using this context.

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;
use Doppar\AI\Vector\Vector;

// 1. Create your Doppar AI agent for OpenAI
$agent = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'));

// 2. Load your precomputed document vectors from local cache
//    The JSON should contain an array of ['vector' => [...], 'content' => '...'] entries
$docs = json_decode(file_get_contents(__DIR__ . '/vector_docs_cache.json'), true);

// 3. Define your question
$question = 'How to create a new OpenAI Doppar AI ?';

// 4. Create an embedding vector for the question
$questionVector = Vector::embedding($agent, 'text-embedding-3-small', $question);

// 5. Retrieve the most relevant context from your docs
$context = Vector::getContext($docs, $questionVector);

// 6. Classic Doppar Agent call with retrieved context
$response = $agent->model('gpt-4o-mini')
    ->messages([
        ['role' => 'system', 'content' => 'You are a helpful Doppar assistant based on a context.'],
        ['role' => 'user', 'content' => "context:\n$context\nQuestion:$question"],
    ])
    ->send();

echo $response;
```

This pattern allows you to build powerful knowledge assistants on top of your own markdown documentation or any text corpus, fully powered by Doppar AI.

### Quantized vs Non-Quantized Models
Quantized models are smaller and faster but may have slightly reduced accuracy. For Pipeline tasks, always use quantized models in production:
```php
// Quantized (faster, smaller)
$result = Pipeline::execute(
    task: TaskEnum::TEXT_GENERATION,
    messages: $messages,
    quantized: true
);

// Non-quantized (more accurate, larger)
$result = Pipeline::execute(
    task: TaskEnum::TEXT_GENERATION,
    messages: $messages,
    quantized: false
);
```

## Rate Limiting for Agents
To prevent excessive usage and control costs when using cloud-based LLMs, implement rate limiting for AI requests. This ensures that a user or system cannot exceed a defined number of requests per time window.
```php
$key = 'ai-agent:' . auth()->id();

if (throttle()->tooManyAttempts($key, 10)) {
    $seconds = throttle()->availableIn($key);
    return response()->json([
        'error' => "Too many requests. Try again in {$seconds} seconds."
    ], 429);
}

throttle()->hit($key, 60); // 10 requests per minute

$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-4')
    ->prompt($userInput)
    ->send();
```

Doppar AI brings powerful machine learning and language model capabilities to your PHP applications with a simple, elegant API. Whether you're analyzing sentiment, generating content, classifying images, or building conversational interfaces, Doppar AI makes it accessible and production-ready.