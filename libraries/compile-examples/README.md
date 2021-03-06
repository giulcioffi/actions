# libraries/compile-examples action

This action compiles all of the examples contained in the library.

## Inputs

### `cli-version`

The version of `arduino-cli` to use. Default `"latest"`.

### `fqbn`

The fully qualified board name to use when compiling. Default `"arduino:avr:uno"`.
For 3rd party boards, also specify the Boards Manager URL:
```yaml
  fqbn: '"sandeepmistry:nRF5:Generic_nRF52832" "https://sandeepmistry.github.io/arduino-nRF5/package_nRF5_boards_index.json"'
```

### `platforms`

YAML-format list of platform dependencies to install.

Default `""`. If no `platforms` input is provided, the board's dependency will be automatically determined from the `fqbn` input and the latest version of that platform will be installed via Board Manager.

If a platform dependency from a non-Board Manager source of the same name as another Board Manager source platform dependency is defined, they will both be installed, with the non-Board Manager dependency overwriting the Board Manager platform installation. This permits testing against a non-release version of a platform while using Board Manager to install the platform's tools dependencies. 
Example:
```yaml
platforms: |
  # Install the latest release of Arduino SAMD Boards and its toolchain via Board Manager
  - name: "arduino:samd"
  # Install the platform from the root of the repository, replacing the BM installed platform
  - source-path: "."
    name: "arduino:samd"
```

#### Sources:

##### Board Manager

Keys:
- `name` - platform name in the form of `VENDOR:ARCHITECTURE`.
- `version` - version of the platform to install. Default is the latest version.

##### Local path

Keys:
- `source-path` - path to install as a platform. Relative paths are assumed to be relative to the root of the repository.
- `name` - platform name in the form of `VENDOR:ARCHITECTURE`.

##### Repository

Keys:
- `source-url` - URL to clone the repository from. It must start with `git://` or end with `.git`.
- `version` - [Git ref](https://git-scm.com/book/en/v2/Git-Internals-Git-References) of the repository to checkout. The special version name `latest` will cause the latest tag to be used. By default, the repository will be checked out to the tip of the default branch.
- `source-path` - path to install as a library. Paths are relative to the root of the repository. The default is to install from the root of the repository.
- `name` - platform name in the form of `VENDOR:ARCHITECTURE`.

##### Archive download

Keys:
- `source-url` - download URL for the archive (e.g., `https://github.com/arduino/ArduinoCore-avr/archive/master.zip`).
- `source-path` - path to install as a library. Paths are relative to the root folder of the archive, or the root of the archive if it has no root folder. The default is to install from the root folder of the archive.
- `name` - platform name in the form of `VENDOR:ARCHITECTURE`.

### `libraries`

YAML-format list of library dependencies to install.

Default `"- source-path: ./"`. This causes the repository to be installed as a library. If there are no library dependencies and you want to override the default, set the `libraries` input to an empty list (`- libraries: '-'`).

Note: the original space-separated list format is also supported. When this syntax is used, the repository under test will always be installed as a library.

#### Sources:

##### Library Manager

Keys:
- `name` - name of the library.
- `version` - version of the library to install. Default is the latest version.

##### Local path

Keys:
- `source-path` - path to install as a library. Relative paths are assumed to be relative to the root of the repository.
- `destination-name` - folder name to install the library to. By default, the folder will be named according to the source repository or subfolder name.

##### Repository

Keys:
- `source-url` - URL to clone the repository from. It must start with `git://` or end with `.git`.
- `version` - [Git ref](https://git-scm.com/book/en/v2/Git-Internals-Git-References) of the repository to checkout. The special version name `latest` will cause the latest tag to be used. By default, the repository will be checked out to the tip of the default branch.
- `source-path` - path to install as a library. Paths are relative to the root of the repository. The default is to install from the root of the repository.
- `destination-name` - folder name to install the library to. By default, the folder will be named according to the source repository or subfolder name.

##### Archive download

Keys:
- `source-url` - download URL for the archive (e.g., `https://github.com/arduino-libraries/Servo/archive/master.zip`).
- `source-path` - path to install as a library. Paths are relative to the root folder of the archive, or the root of the archive if it has no root folder. The default is to install from the root folder of the archive.
- `destination-name` - folder name to install the library to. By default, the folder will be named according to the source archive or subfolder name.

### `sketch-paths`

List of paths containing sketches to compile. These paths will be searched recursively. Default `"examples"`.

### `verbose`

Set to true to show verbose output in the log. Default `false`

### `size-report-sketch`

Name of the sketch used to compare memory usage change. Default `""`.

### `sketches-report-path`

Path in which to save a JSON formatted file containing data from the sketch compilations. Should be used only to store reports. Relative paths are relative to [`GITHUB_WORKSPACE`](https://help.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables). The folder will be created if it doesn't already exist. This report is used by the `arduino/libraries/report-size-deltas` and `arduino/libraries/report-size-trends` actions. Default `"size-deltas-reports"`.

### `github-token`

GitHub access token used to get information from the GitHub API. Only needed if you're using the size report features with private repositories. It will be convenient to use [`${{ secrets.GITHUB_TOKEN }}`](https://help.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token). Default `""`.

### `enable-size-deltas-report`

Set to `true` to cause the action to determine the change in memory usage for the [`size-reports-sketch`](#size-reports-sketch) between the pull request branch and the tip of the pull request's base branch. This may be used with the [`arduino/actions/libraries/report-size-deltas` action](https://github.com/arduino/actions/tree/master/libraries/report-size-deltas). Default `false`.

## Example usage

Only compiling examples:
```yaml
- uses: arduino/actions/libraries/compile-examples@master
  with:
    fqbn: 'arduino:avr:uno'
    libraries: |
      - name: Servo
      - name: Stepper
        version: 1.1.3
```

Storing the memory usage change report as a [workflow artifact](https://help.github.com/en/actions/configuring-and-managing-workflows/persisting-workflow-data-using-artifacts):
```yaml
- uses: arduino/actions/libraries/compile-examples@master
  with:
    size-report-sketch: Foobar
    enable-size-deltas-report: true
- if: github.event_name == 'pull_request'
  uses: actions/upload-artifact@v1
  with:
    name: size-deltas-reports
    path: size-delta-reports
```
