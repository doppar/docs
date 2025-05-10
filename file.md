## File Uploads
### Introduction
Doppar's filesystem configuration file is located at `config/filesystem.php`. Within this file, you may configure all of your filesystem "disks". Each disk represents a particular storage driver and storage location. Example configurations for each supported driver are included in the configuration file so you can modify the configuration to reflect your storage preferences and credentials.

You may configure as many disks as you like and may even have multiple disks that use the same driver. But if you change any of your configuration, and that is not wokring, please clean the application configuration by running the command `config:clear`.

### The Local Driver
When using the local driver, all file operations are relative to the root directory defined in your filesystems configuration file. By default, this value is set to the storage/app/profile directory. Therefore, the following method would write to `storage/app/profile/example.txt`.
```php
use Phaseolies\Support\Facades\Storage;

Storage::disk('local')->store('profile', $request->file('file'));
```

### The Public Disk
The public disk included in your application's filesystems configuration file is intended for files that are going to be publicly accessible. By default, the public disk uses the local driver and stores its files in `storage/app/public`.

If your public disk uses the local driver and you want to make these files accessible from the web, you should create a symbolic link from source directory `storage/app/public` to target directory public/storage:

To create the symbolic link, you may use the `storage:link` Pool command
```bash
php pool storage:link
```

### Display The Image
Once a file has been stored and the symbolic link has been created, you can create a URL to the files using the enqueue helper:
```php
echo enqueue('storage/image.png');
```

### File Upload
To upload file, you can use `Phaseolies\Support\Facades\Storage` facades or you can use direct file object. To upload a image using Storage facades
```php
use Phaseolies\Support\Facades\Storage;

Storage::disk('public')->store('profile', $request->file('file'));
```

This will store file into `storage/app/public/profile` directory. here profile is the directory name that is optional. If you do not pass profile, it will storage file to its default path like `storage/app/public`. You can also uploads file into local disk that is a private directory.
```php
use Phaseolies\Support\Facades\Storage;

Storage::disk('local')->store('profile', $request->file('file'));
```
This will store file into `storage/app/profile` directory. here profile is the directory name that is optional. If you do not pass profile, it will storage file to its default path like `storage/app`. You can also uploads file into local disk that is a private directory.

### Upload with custom file name
If you want to upload file using your customize file name, then remember, `store()` third parameter as $fileName. see the example
```php
$file = uniqid() . '_' . $request->file('file')->getClientOriginalName();

Storage::disk('local')->store('profile', $request->file('file'), $file);
```

### Get the uploaded File
Now if you want to get the uploaded file path, simply you can use get() method
```php
return Storage::disk('local')->get('profile.png'); // it will return the file path
```

### Get the uploaded file contents
To get the uploaded file contents, you can call contents() method, like
```php
Storage::disk('local')->content('product/product.json');

response()->file($pathToFile); // You can use this also
```

### Delete file
To delete file from storage disk, you can call delete() method
```php
return Storage::disk('local')->delete('product/product.json');
```

### Upload File Without Storage
You can upload file any of your application folder. Doppar allows it also. To upload file without Storage facades,
```php
$request->file('file')->store('product');
```
Now your file will be stored in profuct directory inside storage folder. You can also use the storeAs method by passing your custom file name. You can pass a callback with storeAs method like
```php
$admin = 0;
$request->file('file')->storeAs(function ($file) use ($admin) {
    if (! $admin) {
        return true;
    }
}, 'product-cart', 'file_name');
```

You can also use move method to upload your file.

```php
$file = $request->file('invoice');
$file->move($destinationPath, $fileName)
```

### File Downloads
The download method generates a response that triggers a file download in the user’s browser. It takes the file path as its primary argument. Optionally, you can specify a custom download filename as the second argument—this overrides the default name seen by the user. Additionally, an array of custom HTTP headers can be passed as a third argument for further control over the download behavior.
```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

### File Responses
The file method may be used to display a file, such as an image or PDF, directly in the user's browser instead of initiating a download. This method accepts the absolute path to the file as its first argument and an array of headers as its second argument:

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

### Streamed Downloads
At times, you may want to convert the string output of an operation into a downloadable response without storing it on disk. The streamDownload method allows you to achieve this by accepting a callback, filename, and an optional array of headers as parameters:
```php

use App\Services\Doppar;

return response()->streamDownload(function () {
    echo Doppar::api('repo')
        ->contents()
        ->readme('doppar', 'doppar')['contents'];
}, 'doppar-readme.md');
```

