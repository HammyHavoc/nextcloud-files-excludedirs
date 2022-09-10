# Exclude Directories Plugin for Nextcloud
Excludes directories and files from being added to the Nextcloud file cache.

- **For files not yet in Nextcloud that match an ignored path:** The files will
  be skipped and will not appear in Nextcloud after future file scans.
- **For files already in Nextcloud that match an ignored path:** The files will
  appear in search results, but users will be blocked from accessing them. If a
  user attempts to open a blocked directory, they will be blocked from creating
  new files under the directory. Users are also blocked from creating any file
  or folder that matches any of the rules.

## Compatibility
Should be compatible up to Nextcloud 23.x.

## Installation
### From Package
1. Download a package from the Releases page of this GitHub page.
2. Expand the package using `tar -xvzf nextcloud-files-excludedirs-vVERSION.tar.gz`.
3. Move the expanded `files_excludedirs` folder into your `nextcloud/apps` 
   directory, such that `composer.json` can be found as 
   `nextcloud/apps/files_excludedirs/composer.json`.
4. Enable the "Exclude directories" plugin under the "Apps" admin page
   (`/settings/apps`).

### From Source
1. Ensure that your system has Git and Composer installed.
2. Clone this repository into a new subdirectory under your `nextcloud/apps`
   directory, and name the folder `files_excludedirs`.
3. In the `files_excludedirs` subdirectory, run `composer install` to install
   required dependencies.
4. Ensure the `files_excludedirs` subdirectory and all its subdirectories are
   owned by a user that the Nextcloud user can read (e.g., if NC user is in the
   `www-data` group, the folder should be owned by `root:www-data` with dir
   permissions set to `0755` and files set to `0644`).

## Configuring Excluded Path Patterns
The default rule is to exclude paths that match `[".snapshot"]` (i.e., to
exclude Btrfs snapshot folders from being indexed). 

There is no settings page (yet) for this plugin, but the default rule can be
overridden using the Nextcloud CLI using a command like the following:

```
occ config:app:set files_excludedirs exclude \
    --value '[".snapshot","anotherfolder", "pattern/*/match"]'
```

This will exclude:
- All folders named `.snapshot`.
- All folders named `anotherfolder`.
- Folders that match the glob pattern `pattern/*/match`, including 
  `pattern/xyz/match` and `pattern/abc/match`.

## Removing Unwanted Existing Files from Nextcloud File Cache
Even with this plugin enabled, **neither** a file scan (`occ files:scan --all`) nor
file cleanup (`occ files:cleanup`) **is sufficient** to remove excluded paths
from the file cache. Instead, you will need to remove these files from the
database using SQL like the following:

```mariadb
DELETE FROM filecache WHERE filecache.path LIKE '%.snapshot%'
```

## Credits
Initially authored by Roeland Jago Douma, with contributions from:
- Robin Appelman
- King
- Alan J. Pippin
- Guy Elsmore-Paddock
