<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

<p align="center">
<a href="https://github.com/laravel/framework/actions"><img src="https://github.com/laravel/framework/workflows/tests/badge.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

### Create Laravel Project
``` 
composer create-project laravel/laravel example-app
```

### Run project 
``` 
php artisan serve
```

### Install Laravel Spatie Backup Package

``` 
composer require spatie/laravel-backup
```
To publish the config file to config/backup.php run:
```
php artisan vendor:publish --provider="Spatie\Backup\BackupServiceProvider"
```

### Add this for Xampp in app/config/database.php

``` 
'connections' => [

        'mysql' => [
            'driver' => 'mysql',
            'dump' => [
                'dump_binary_path' => 'C:/xampp/mysql/bin/', // only the path, so without `mysqldump` or `pg_dump`
                'use_single_transaction',
                'timeout' => 60 * 5, // 5 minute timeout
            ],
        ],
    ],
```
### Configure your .env file for sending Notificaion
``` 
MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

### Taking backups
``` 
php artisan backup:run
```
If you only need to backup the db, run:
``` 
php artisan backup:run --only-db
```
If you only need to backup the files, and want to skip dumping the databases, run:
``` 
php artisan backup:run --only-files
```
### Cleaning up old backups
``` 
php artisan backup:clean
```

# Backup Laravel Project to Google Drive

Install Google drive filesystem extension
``` 
composer require masbug/flysystem-google-drive-ext
```

### Configure .env file
```
GOOGLE_DRIVE_CLIENT_ID=
GOOGLE_DRIVE_CLIENT_SECRET=xxx
GOOGLE_DRIVE_REFRESH_TOKEN=xxx
GOOGLE_DRIVE_FOLDER=Backups
```

### Add disks on config/filesystems.php
``` 
'google' => [
        'driver' => 'google',
        'clientId' => env('GOOGLE_DRIVE_CLIENT_ID'),
        'clientSecret' => env('GOOGLE_DRIVE_CLIENT_SECRET'),
        'refreshToken' => env('GOOGLE_DRIVE_REFRESH_TOKEN'),
        'folder' => env('GOOGLE_DRIVE_FOLDER'), 
    ],
```

### Add driver storage in a ServiceProvider on path app/Providers/
``` 
public function boot(){
   
        try {
            \Storage::extend('google', function($app, $config) {
                $options = [];

                if (!empty($config['teamDriveId'] ?? null)) {
                    $options['teamDriveId'] = $config['teamDriveId'];
                }

                $client = new \Google\Client();
                $client->setClientId($config['clientId']);
                $client->setClientSecret($config['clientSecret']);
                $client->refreshToken($config['refreshToken']);
                
                $service = new \Google\Service\Drive($client);
                $adapter = new \Masbug\Flysystem\GoogleDriveAdapter($service, $config['folder'] ?? '/', $options);
                $driver = new \League\Flysystem\Filesystem($adapter);

                return new \Illuminate\Filesystem\FilesystemAdapter($driver, $adapter);
            });
        } catch(\Exception $e) {
            // your exception handling logic
        }
       
    }
```

### Getting Google Keys
<ul>
<li>
    <a href="https://console.cloud.google.com/">Getting your Client ID and Secret</a>
</li>
<li>
<a href="https://github.com/ivanvermeyen/laravel-google-drive-demo/blob/master/README/2-getting-your-refresh-token.md">Getting your Refresh Token</a>
</li>
</ul>

### Update app/config/backup.php
``` 
 'destination' => [
 
            'disks' => [
                // 'local', change local to google
                'google',
            ],
        ],
```

### Create a folder named "Backups" in your google drive. Then create another folder named "Laravel" into this Backups folder. This "Laravel" name should be same as storage/app/Laravel
<ul>
<li>
    <a href="https://drive.google.com/">Google Drive</a>
</li>
</ul>

### Run command for backup
``` 
php artisan backup:run
```
