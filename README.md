# AlbumArt

This repo was created to preserve a Python3 script `albumart.py` written by Jean-Francois Dokes (JFD) to leverage his [Libupnpp for Control Points](https://www.lesbonscomptes.com/upmpdcli/libupnpp-refdoc/libupnpp-ctl.html) and made available by him to the [moOde Audio Player](https://github/moode-player/moode) project.

The script was found to be missing a `debug()` function, which was added in my first commit after checking in the original.

The script is intended to be used in moOde as a drop-in replacement for `upexplorer`, another utility written by JFD in C++. The latter utility leverages JFD's libupnpp functionality to fetch information including, optionally, the albumArtURI from a UPnP/AV renderer playing a track. It worked well in this role but was unable to do the same for an OpenHome renderer. The `albumart.py` script deals solely with the albumArtURI from either type of renderer.

The backend code in moOde is written primarily in PHP. It invokes `upexplorer` through a PHP `exec()` function call as shown in the following code snippet

```
// get upnp coverart url
function getUpnpCoverUrl() {
        $result = sysCmd('upexplorer --album-art "' . $_SESSION['upnpname'] . '"');
        return $result[0];
```
where the session variable contains the friendly name of the renderer being queried, for example, 'Moode UPNP' and `sysCmd` is defined as
```
function sysCmd($cmd) {
        exec('sudo ' . $cmd . " 2>&1", $output);
        return $output;
```
In principle, it should have been possible to replace `upexplorer --album-art` with `albumart.py` and drive on. In practice, the albumArtURI was not being returned to PHP even though `albumart.py` worked flawlessly when invoked from the command line. Using `sys_exec()` in place of `exec()` with appropriate changes made no difference.

The culprit turned out to be the use of `os._exit()` in the Python script. When replaced with `sys.exit()` the script output gets returned. Changing the exit call was the subject of my next commit.

Possible future change: integrate logging with the existing moOde/upmpdcli logging scheme.

