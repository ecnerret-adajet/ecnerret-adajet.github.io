---
layout: single
title:  "Import Excel data to DB using Vue and Laravel"
categories:
  - Web
tags:
  - import
  - excel
  - vue
  - Laravel
  - maatwebsite
---
This is a simple guide on how we can import excel or csv file and store the content directly to database using `maatwebsite` pacakge with `Vuejs` on Laravel version 5.8 to 7 as of this writing.

For the basic part, we will not include here the installation of laravel project. I assumed that you already have laravel ready.

### 1. Install maatwebsite pacakge
First, we will install the package via composer dependency manager. open a command-line, navigate to your project path and run the following command below.
```yaml
composer require maatwebsite/excel
```
After installing the package. go to your `config/app.php` file and add these config to `provider` and `aliase` section.
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
Now let's create a dedicated Import class to handle the data on our excel file.
```yaml
php artisan make:import UsersImport --model=User
```
Next, navigate to to `App\Imports\UsersImport.php`. The excel file  header should contain the following object based from the `UserImport` return function.
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
If you like to know more about what other options available from this pacakge. you may visit the official documentation page.

**Note:** https://docs.laravel-excel.com/
{: .notice--info}


### 3. Import Backend Setup
Here's the working example on we can import a excel file to our backend controller.
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
Then, open your `web.php` or `api.php` to set-up the route link for our import function.
```php
<!-- routes/web.php or routes/api -->
Route::post('/users/import','UsersController@import');
```
### 3. Front-End Setup
Now that we prepared the backend function to process the excel file. we will now proceed to our vuejs section. From here, you may notice that we used `axios` to call the route link that we setup from `step 3`. you may extend this simple component whatever you want.
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
### 4. Compile Project.
Finally, compile the project to load our js files by running `npm` command. Open up again your command-line and run the script below.
```yaml
npm run dev
```
And that's all. open now a browser to test the program.

Happy Coding.

