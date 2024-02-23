#!/usr/bin/swift

import Foundation

let versionNumber: String = "1.0"
let defaultSpaceThresholdGB: Double = 10.0

struct Config {
    var isQuietMode: Bool = false
    var isDryRun: Bool = false
    var spaceThresholdGB: Double = defaultSpaceThresholdGB
    var trashDirectoryPath: String?
}

func main() {
    let config = processCommandLineArguments()
    
    let trashDirectoryPath = config.trashDirectoryPath ?? FileManager.default.urls(for: .trashDirectory, in: .userDomainMask).first!.path
    let spaceThresholdBytes = convertGBToBytes(config.spaceThresholdGB)
    let availableSpaceBeforeCleanup = fetchAvailableDiskSpace(trashDirectoryPath)
    var totalSizeRemoved: Int64 = 0

    let filesInTrash = fetchFilesInTrashSortedByDateAdded(trashDirectoryPath)
    for fileURL in filesInTrash {
        let currentAvailableSpace = fetchAvailableDiskSpace(trashDirectoryPath)
        if currentAvailableSpace >= spaceThresholdBytes { break }
        let fileSize = calculateFileSize(fileURL)
        if !config.isDryRun {
            removeFile(fileURL)
        }
        totalSizeRemoved += fileSize
        printRemovalMessage(fileSize, fileURL, isQuietMode: config.isQuietMode)
    }

    let availableSpaceAfterCleanup = fetchAvailableDiskSpace(trashDirectoryPath)
    printFinalReport(availableSpaceBeforeCleanup, totalSizeRemoved, availableSpaceAfterCleanup, isQuietMode: config.isQuietMode)
}

func processCommandLineArguments() -> Config {
    var config = Config()
    let args = CommandLine.arguments[1...]
    
    if args.contains("-h") || args.contains("--help") {
        printUsage()
        exit(0)
    }
    if args.contains("-v") || args.contains("--version") {
        printVersion()
        exit(0)
    }
    if args.contains("-q") || args.contains("--quiet") {
        config.isQuietMode = true
    }
    if args.contains("-d") || args.contains("--dry-run") {
        config.isDryRun = true
    }
    if let thresholdIndex = args.firstIndex(where: { $0 == "-t" || $0 == "--threshold" }) {
        if thresholdIndex + 1 <= args.count, let threshold = Double(args[args.index(after: thresholdIndex)]) {
            config.spaceThresholdGB = threshold
        }
    }
    if let pathIndex = args.firstIndex(where: { $0 == "-p" || $0 == "--path" }) {
        if pathIndex + 1 <= args.count {
            config.trashDirectoryPath = args[args.index(after: pathIndex)]
        }
    }
    return config
}

func printUsage() {
    let scriptName = getScriptName()
    let usageText = """
    Usage: \(scriptName) [OPTION]...
    Automatically cleans up files in the specified path if the available disk space
    is below the specified threshold (in GB, default is \(defaultSpaceThresholdGB) GB). The utility selects items
    based on their date added, starting with the oldest, and
    stops deleting when the available disk space reaches the threshold.
    The default path is the user's Trash directory.
    
    Options:
      -d, --dry-run    simulate the cleanup process without deleting files
      -h, --help       display this help and exit
      -p, --path       set the path to clean (defaults to Trash)
      -q, --quiet      suppress all output
      -t, --threshold  set the disk space threshold in GB (default is \(defaultSpaceThresholdGB) GB)
      -v, --version    display version information and exit
      
    Example:
      \(scriptName) --threshold 5 --path /path/to/clean
                       Cleans up the specified path if the available disk space is
                       less than 5 GB.
    """
    print(usageText)
}

func printVersion() {
    let scriptName = getScriptName()
    print("\(scriptName) version \(versionNumber)")
}

func getScriptName() -> String {
    return CommandLine.arguments[0].split(separator: "/").last.map(String.init) ?? "trashsweeper"
}

func fetchAvailableDiskSpace(_ trashDirectoryPath: String) -> Int64 {
    do {
        let fileSystemAttributes = try FileManager.default.attributesOfFileSystem(forPath: trashDirectoryPath)
        if let availableSpace = fileSystemAttributes[.systemFreeSize] as? NSNumber {
            return availableSpace.int64Value
        } else {
            fatalError("Could not find available disk space attribute.")
        }
    } catch {
        fatalError("Error fetching available disk space: \(error.localizedDescription)")
    }
}

func fetchFilesInTrashSortedByDateAdded(_ trashDirectoryPath: String) -> [URL] {
    do {
        let trashDirectoryURL = URL(fileURLWithPath: trashDirectoryPath)
        let sortingKeys: Set<URLResourceKey> = [.addedToDirectoryDateKey]
        let filesInTrash = try FileManager.default.contentsOfDirectory(at: trashDirectoryURL, includingPropertiesForKeys: Array(sortingKeys), options: [])
        return filesInTrash.compactMap { fileURL -> (url: URL, date: Date)? in
            do {
                let dateAdded = try fileURL.resourceValues(forKeys: sortingKeys).addedToDirectoryDate
                return dateAdded != nil ? (fileURL, dateAdded!) : nil
            } catch {
                fatalError("Error retrieving date added for file \(fileURL.lastPathComponent): \(error.localizedDescription)")
            }
        }
        .sorted { $0.date < $1.date }
        .map { $0.url }
    } catch {
        fatalError("Error fetching files in trash sorted by date added: \(error.localizedDescription)")
    }
}

func calculateFileSize(_ fileURL: URL) -> Int64 {
    do {
        let fileManager = FileManager.default
        let resourceValues = try fileURL.resourceValues(forKeys: [.isDirectoryKey, .fileSizeKey])
        if let isDirectory = resourceValues.isDirectory, isDirectory {
            var totalSize: Int64 = 0
            let fileEnumerator = fileManager.enumerator(at: fileURL, includingPropertiesForKeys: [.fileSizeKey], options: [], errorHandler: nil)!
            for case let fileURL as URL in fileEnumerator {
                let fileAttributes = try fileURL.resourceValues(forKeys: [.fileSizeKey])
                if let fileSize = fileAttributes.fileSize {
                    totalSize += Int64(fileSize)
                }
            }
            return totalSize
        } else if let fileSize = resourceValues.fileSize {
            return Int64(fileSize)
        } else {
            return 0
        }
    } catch {
        fatalError("Error calculating file size for \(fileURL.lastPathComponent): \(error.localizedDescription)")
    }
}

func removeFile(_ fileURL: URL) {
    do {
        try FileManager.default.removeItem(at: fileURL)
    } catch {
        fatalError("Error removing file \(fileURL.lastPathComponent): \(error.localizedDescription)")
    }
}

func convertGBToBytes(_ gigabytes: Double) -> Int64 {
    return Int64(gigabytes * 1024 * 1024 * 1024)
}

func convertBytesToReadableFormat(_ bytes: Int64) -> String {
    let units = ["B", "KB", "MB", "GB", "TB"]
    var size = Double(bytes)
    var unitIndex = 0
    while size >= 1024 && unitIndex < units.count - 1 {
        size /= 1024
        unitIndex += 1
    }
    return String(format: "%.2f \(units[unitIndex])", size)
}

func printRemovalMessage(_ fileSize: Int64, _ fileURL: URL, isQuietMode: Bool) {
    if !isQuietMode {
        print("\(convertBytesToReadableFormat(fileSize))\tRemoved: \(fileURL.lastPathComponent)")
    }
}

func printFinalReport(_ before: Int64, _ removed: Int64, _ after: Int64, isQuietMode: Bool) {
    if !isQuietMode {
        print("\n-------------------------\n")
        print("Before cleanup: \(convertBytesToReadableFormat(before))")
        print("Total removed: \(convertBytesToReadableFormat(removed))")
        print("After cleanup: \(convertBytesToReadableFormat(after))")
    }
}

main()

