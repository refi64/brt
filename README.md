# brt

A binary asset retrieval tool.

## Features

- Automatically saves the sha256 hashes of the files in a lock file (brtlock.json),
  thus ensuring reproducibility without you having to manually track the URLs.

## Requirements

- Tcl 8.5+
- [jsonnet](https://jsonnet.org/)
- p7zip to extract zip files
- bsdtar to extract any other archive formats

## Usage

Create a `brtfile.jsonnet` (see below), then run `brt` to download everything. There
are no command line options at the moment.

Note that, while old downloads are deleted, the symlinks brt creates must be deleted
manually.

## brtfile format

```jsonnet
{
  files: [
    /* Define a file to download. This will be saved to foo/bar
       (foo will be created if it does not already exist).

       Note that the file will be saved into .brt/, and a symlink will be created at the
       path you give. */
    {
      url: 'https://mysite/the-file',
      dest: 'test1/a-file'
    },
    /* If dest ends with a /, then the last component of the url is used as the filename.
       This will be saved to test2/some-name: */
    {
      url: 'https://mysite/some-name',
      dest: 'test2/'
    },
    /* You can add a post command to run after the download. Note that you must NOT
       edit the downloaded file, otherwise the hash will be shown as wrong and brt
       will re-download it next time.

       The command is run from the current directory, and $BRT_DOWNLOAD will be set to
       the path to the downloaded file. */
    {
      url: 'https://mysite/something-else',
      dest: 'test3/',
      post: 'echo test3 is saved at: $BRT_DOWNLOAD'
    },
    /* Archive extraction works by setting arc: true. This will extract the contents of arc1
       to be inside test3/arc1/, e.g. a file x/y in arc1.tar.gz will become tet3/arc1/x/y. */
    {
      url: 'https://mysite/arc1.tar.gz',
      dest: 'test4/arc1',
      arc: true
    },
    /* If the files inside the archive have a common parent, you can strip it off. This will
       extract arc2.tar.gz, but the prefix dir/ will be stripped off every path. Thus,
       dir/x/y/z will be saved as test5/arc2/x/y/z, *not* test5/arc2/dir/x/y/z. */
    {
      url: 'https://mysite/arc2.tar.gz',
      dest: 'test5/arc2',
      arc: {prefix: 'dir'}
    },
    /* Just like above, you can run commands after extraction. Note that the post key shown
       above will run *before* extraction, but this will run after. $BRT_EXTRACTED will be
       set to the directory holding the extracted files, *before any prefixes are removed*.
       You can do this to e.g. rename the directory to have a common prefix.

       Note that, unlike in the regular post, arc.post lets you modify the files inside the
       extracted directory. */
    {
      url: 'https://mysite/arc3.tar.gz',
      dest: 'test6/arc3',
      arc: {
        post: 'echo Archive contents are currently in: $BRT_EXTRACTED'
      }
    },
    /* Since this is jsonnet, you can use all the fancy syntax you want to make this file
       less repetitive. */
  ]
}
```

## TODO

- Re-run downloads and extraction on post script changes.
- Remove dead symlinks.
- Probably rewrite in Go or C/C++ or something to use the direct jsonnet APIs.
