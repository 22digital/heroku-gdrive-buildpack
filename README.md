# Heroku Google Drive Buildpack

Remote [Google Drive client][heroku-drive] on Heroku using `rclone` & `WinRAR`.
Optionally you can download files using `Aria2`.

## Installation
Create new app

```bash
$ heroku create myapp -b https://github.com/thecreativeacademy/heroku-google-drive.git
$ heroku git:clone -a myapp
```

Existing app, use: `add|set`.

```bash
$ heroku buildpacks:set https://github.com/thecreativeacademy/heroku-google-drive.git -a myapp
```

Go to `myapp` directory, create or copy `rclone.conf` and WinRAR registraton 
key `.rarreg.key` (optional) then commit the change.

```bash
$ cd myapp
$ git add .
$ git commit -am "add config"
$ git push heroku master
```

If you don't have `rclone.conf` download `rclone` and run `rclone config` 
locally to generate a config file. This file will be found at:

```text
Windows: %userprofile%\.config\rclone\rclone.conf
Linux/macOS: $HOME/.config/rclone/rclone.conf
```

## Usage

### Open a remote Heroku connection

```bash
$ cd myapp
$ heroku run bash
```

**or:**

```
$ heroku run bash --remote origin
```

### Upload to Google Drive

Assume `gdrive_config` is your Google drive config name that was generated 
when creating the config file. 

```bash
$ rclone -v copy local_dir gdrive_config:remote_drive_dir
```

### Speed up uploads

If you want to upload files smaller than 8mb increase the `--transfers` option.

```bash
$ rclone -v --transfers=16 --drive-chunk-size=16384k --drive-upload-cutoff=16384k copy local_dir gdrive_config:remote_drive_dir
 ```
 
`--transfers=N` number parallel of connection (`default: 4`).

` --drive-chunk-size=N` if file bigger than this size it will split into 
multiple uploads. Increase if you want better speed (`default: 8192k or 8mb`).

`--drive-upload-cutoff=N` should be same with chunk size.

`-v` option to view upload progress stats.

### View file on Google drive

```bash
$ rclone lsd gdrive_config:remote_drive_dir
```

View options:

- `lsd` only show file in current directory.
- `ls` show file including in subdirectory (recursively).

## Bonus

### Download file using `Aria2`

Aria2 is a command-line download accelerator.

#### 231 MB Sample File

```bash
$ aria2c -x16 https://s3-eu-west-1.amazonaws.com/pfigshare-u-files/9491437/standard.rar
```

#### 617 MB Sample File

```bash
$ aria2c -x16 https://s3-eu-west-1.amazonaws.com/pfigshare-u-files/9491434/sample.rar
```

`-x16` sets the total number of connections to the server.

### To extract `.rar` file

In current directory:

```bash
$ unrar e standard.rar
```

With full path:

```bash
$ unrar x standard.rar
```


[heroku-drive]: https://github.com/thecreativeacademy/heroku-google-drive
