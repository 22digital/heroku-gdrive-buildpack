# Heroku Google Drive Buildpack

Remote [Google Drive client][heroku-drive] on Heroku using `rclone` to 
sync/upload/download files to. Optionally you can download files using `Aria2` 
and unpack/pack `.rar` files using `WinRAR`. There is also an installation of
`Git LFS` for usage in large filesystem git repositories.

<!-- MarkdownTOC -->

- [Applications](#applications)
- [Installation](#installation)
- [Binaries](#binaries)
- [Configure Rclone Remote](#configure-rclone-remote)
    - [SSH Keys](#ssh-keys)
- [WinRAR License Key](#winrar-license-key)
- [Usage](#usage)
    - [Open a remote Heroku connection](#open-a-remote-heroku-connection)
    - [Upload to Google Drive](#upload-to-google-drive)
    - [Speed up uploads](#speed-up-uploads)
    - [View file on Google drive](#view-file-on-google-drive)
- [Bonus](#bonus)
    - [Download file using `Aria2`](#download-file-using-aria2)
        - [231 MB Sample File](#231-mb-sample-file)
        - [617 MB Sample File](#617-mb-sample-file)
    - [To extract `.rar` file](#to-extract-rar-file)

<!-- /MarkdownTOC -->


## Applications

The following apps are installed alongside this buildback:

1. [Aria2][aria]
2. [Git LFS][lfs]
3. [rclone][rclone]
4. [WinRAR][winrar]

## Installation

1). Create new app:

```bash
$ heroku create myapp -b https://github.com/thecreativeacademy/heroku-gdrive-buildpack.git
$ heroku git:clone -a myapp
```

2). Existing app, use: `add|set`.

```bash
$ heroku buildpacks:set https://github.com/thecreativeacademy/heroku-gdrive-buildpack.git -a myapp
```

**NB:** To run this buildpack you **must have** an `rclone.conf` file in the 
root of project. The build pack **will not run** if you do not have this file 
in this location.

3). Go to `myapp` directory and update configuration in `rclone.conf`.
4). Optionally, and a WinRAR registration key in the `.rarreg.key` file.
5). Commit all the changes.

```bash
$ cd myapp
$ git add .
$ git commit -am "Initial commit with config files."
$ git push heroku master
```

## Binaries

This buildpack installs packages to the following locations:

| Application | Path                        |
|:------------|:----------------------------|
| aria2c      | /app/vendor/aria2c/aria2c   |
| git-lfs     | /app/vendor/git-lfs/git-lfs |
| rclone      | /app/vendor/rclone/rclone   |
| rar         | /app/vendor/winrar/rar      |
| unrar       | /app/vendor/winrar/unrar    |

## Configure Rclone Remote

If you don't have an `rclone.conf` file on your machine, download 
[rclone][rclone] and run `rclone config` locally to generate a config file. 
Once created, this file will be found at:

```text
Windows: %userprofile%\.config\rclone\rclone.conf
Linux/macOS: $HOME/.config/rclone/rclone.conf
```

Visit the [Google Drive rclone][rclone-drive] page to view detailed setup 
instructions and all available options you can set when configuring the Google 
Drive remote.

### SSH Keys

If you configure your `rclone` remote to use SFTP or your Google Drive config 
uses an openssh key pair then you can commit your private key to the root 
folder of the repository.

The buildpack will automatically search for a private key file called `id_rsa` 
and, if found in the _project root folder_, will move this file to 
`/app/.ssh/id_rsa`.

## WinRAR License Key

If you use [WinRAR][winrar] you will need to copy the registration key file
`rarreg.key` to the root of your Heroku project folder in order for `rar` and 
`unrar` to work correctly.

More on licensing from WinRAR:

> Now WinRAR, command line RAR for Unix and macOS use the same registration
> key format, so you can use the same key with current WinRAR and RAR versions
> for all mentioned platforms. It is not guaranteed for WinRAR and RAR versions
> that are not equal to version included to this distribution file.
>
> If you use RAR/Unix and RAR for macOS, you should copy rarreg.key
> to your home directory or to one of the following directories:
> /etc, /usr/lib, /usr/local/lib, /usr/local/etc. You may rename it
> to `.rarreg.key` or `.rarregkey`, if you wish, but rarreg.key is also valid.

We recommend changing the filename to `.rarreg.key` when placing it in the root of your project folder.

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

With full path:

```bash
$ unrar x ./standard.rar
```

In current directory (use with caution):

```bash
$ unrar e ./standard.rar
```

[heroku-drive]: https://github.com/thecreativeacademy/heroku-gdrive-buildpack
[rclone-drive]: https://rclone.org/drive/
[aria]: https://aria2.github.io/
[lfs]: https://git-lfs.github.com/
[rclone]: https://rclone.org
[winrar]: https://www.rarlab.com/
