---
layout: single
title:  "Import Excel data to DB using Vue to Laravel"
categories:
  - Web
tags:
  - import
  - excel
  - vue
  - Laravel
  - maatwebsite
---
This is a simple guide on how we can import excel or csv file and store the content directly to database. Using `maatwebsite` pacakge with `Vuejs` on Laravel version 5.8 to 7 as of this writing.

For the basic part, we will not include here the installation and setting-up config of laravel project. I assumed that you already have laravel ready.

### 1. Install maatwebsite pacakge
First, we will install the package via composer dependcy manager. open command-line, navigate to your project path and run the following command below.
```yaml
composer require maatwebsite/excel
```
After installling, go to your `config/app.php` file and the these to `provider` and `aliase` section.
```php
'providers' => [
	/*
  * Package Service Providers...
  */
	Maatwebsite\Excel\ExcelServiceProvider::class,
],
'aliases' => [
	....
	'Excel' => Maatwebsite\Excel\Facades\Excel::class,
],
```
Open again the command-line and run the vendor publish command
```yaml
php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```
### 2. Setting up Imports

```yaml
php artisan make:import UsersImport --model=User
```

```php
<?php

namespace App\Imports;

use App\User;
use Maatwebsite\Excel\Concerns\ToModel;
use Maatwebsite\Excel\Concerns\WithHeadingRow;

class UsersImport implements ToModel, WithHeadingRow
{
    /**
    * @param array $row
    *
    * @return \Illuminate\Database\Eloquent\Model|null
    */
    public function model(array $row)
    {
        return new User([
            'name'  => $row['name'],
            'email' => $row['email'],
            'password' => Hash::make($row['password']),
        ]);
    }
}
```


```php
<!-- UsersController -->
use App\Imports\UsersImport;
use Maatwebsite\Excel\Facades\Excel;
use App\Http\Controllers\Controller;

class UsersController extends Controller
{
    public function import(Request $request)
    {
         $request->validate([
            'import_file' => 'required|file|mimes:xls,xlsx'
        ]);

        $path = $request->file('import_file');
        $data = Excel::import(new UsersImport, $path);

        return response()->json(['message' => 'uploaded successfully'], 200);
    }
}
```

```php
<!-- routes/web.php or routes/api -->
Route::post('/users/import','UsersController@import');
```

### 3. upload file thorugh Vuejs

```javascript
<template>
   <div class="row">
      <div class="form-row">
        <div class="col-md-12">
          <label class="form-control-label"  for="input-file-import">Upload Excel File</label>
          <input type="file" class="form-control" :class="{ ' is-invalid' : error.message }" id="input-file-import" name="file_import" ref="import_file"  @change="onFileChange">
          <div v-if="error.message" class="invalid-feedback">
            {{ error.message }}
          </div>
        </div>
      </div>
   </div>
</template>
<script>
  export default {
     data() {
        return {
          error: {},
          import_file: '',
        }
      },
      methods: {
         onFileChange(e) {
        this.import_file = e.target.files[0];
    },

    proceedAction() {
        let formData = new FormData();
        formData.append('import_file', this.import_file);

          axios.post('/users/import', formData, {
              headers: { 'content-type': 'multipart/form-data' }
            })
            .then(response => {
                if(response.status === 200) {
                  // codes here after the file is upload successfully
                }
            })
            .catch(error => {
                // code here when an upload is not valid
                this.uploading = false
                this.error = error.response.data
                console.log('check error: ', this.error)
            });
        },
      }
  }
</script>
```

