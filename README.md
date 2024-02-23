# TrashSweeper

`trashsweeper` is a command-line utility designed for macOS that helps manage disk space by automatically cleaning up files in a specified directory, typically the Trash. It's particularly useful for automating the process of freeing up disk space when it falls below a certain threshold.

## Features

- **Automatic Cleanup**: Cleans up files in the specified directory based on the available disk space and a user-defined threshold.
- **Customizable Threshold**: Allows users to specify the disk space threshold in GB. The default threshold is set to 10 GB.
- **Custom Path Support**: While the default path is the user's Trash directory, `trashsweeper` can be configured to clean any specified directory.
- **Quiet Mode**: Offers an option to suppress all output for silent operation.
- **Dry Run Option**: Includes a dry run mode that simulates the cleanup process without actually deleting any files, useful for testing.
- **Version Information**: Provides version information of the utility.

## Installation

To use `trashsweeper`, download the script and ensure it has executable permissions:

```bash
chmod +x trashsweeper
```

## Usage

```bash
./trashsweeper [OPTION]...
```

### Options

- `-d`, `--dry-run`: Simulate the cleanup process without deleting files.
- `-h`, `--help`: Display help and exit.
- `-p`, `--path`: Set the path to clean (defaults to Trash).
- `-q`, `--quiet`: Suppress all output.
- `-t`, `--threshold`: Set the disk space threshold in GB (default is 10 GB).
- `-v`, `--version`: Display version information and exit.

### Examples

To clean up the Trash directory if the available disk space is less than 5 GB:

```bash
./trashsweeper --threshold 5
```

To simulate cleaning a custom directory without actually deleting files:

```bash
./trashsweeper --dry-run --path /path/to/directory
```

## Contributing

Contributions to `trashsweeper` are welcome. Please feel free to submit pull requests or open issues to suggest improvements or report bugs.

## License

`trashsweeper` is open-source software licensed under the MIT License. See the LICENSE file for more details.

