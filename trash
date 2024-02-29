#!/usr/bin/env swift

import Foundation

let version = "1.0"

func executeAppleScript(_ script: String) {
    var error: NSDictionary?
    if let scriptObject = NSAppleScript(source: script) {
        let output = scriptObject.executeAndReturnError(&error)
        if let error = error {
            print("Error: \(error)")
        } else {
            print(output.stringValue ?? "")
        }
    }
}

func moveToTrash(fileURL: URL) {
    let fileManager = FileManager.default
    if !fileManager.fileExists(atPath: fileURL.path) {
        print("Error: The file does not exist at path \(fileURL.path)")
        return
    }
    let appleScript = """
    tell application "Finder"
        move POSIX file "\(fileURL.path)" to trash
    end tell
    """
    executeAppleScript(appleScript)
}

func listTrashContents() {
    guard let trashURL = FileManager.default.urls(for: .trashDirectory, in: .userDomainMask).first else {
        print("Unable to locate the Trash directory.")
        return
    }
    do {
        let trashContents = try FileManager.default.contentsOfDirectory(at: trashURL, includingPropertiesForKeys: nil)
        if trashContents.isEmpty {
            print("The Trash is empty.")
        } else {
            for item in trashContents {
                print(item.lastPathComponent)
            }
        }
    } catch {
        print("Error listing Trash contents: \(error.localizedDescription)")
    }
}

func printUsage() {
    let usage = """
    Usage: trash [-h] [-v] [-l] <file path 1> <file path 2> ... <file path N>
    Options:
      -h, --help        Show this help message and exit
      -v, --version     Show program's version number and exit
      -l, --list        List the contents of the Trash
    """
    print(usage)
}

func printVersion() {
    print("trash version \(version)")
}

let args = CommandLine.arguments

if args.contains("-h") || args.contains("--help") {
    printUsage()
    exit(0)
}

if args.contains("-v") || args.contains("--version") {
    printVersion()
    exit(0)
}

if args.contains("-l") || args.contains("--list") {
    listTrashContents()
    exit(0)
}

if args.count < 2 {
    printUsage()
    exit(1)
}

for path in args.dropFirst() {
    if path == "-h" || path == "--help" || path == "-v" || path == "--version" || path == "-l" || path == "--list" {
        continue
    }
    let fileURL = URL(fileURLWithPath: path)
    moveToTrash(fileURL: fileURL)
}
