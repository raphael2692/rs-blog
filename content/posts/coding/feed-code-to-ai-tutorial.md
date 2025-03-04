+++
title = "Feed code efficiently to AI with Code Merger"
tags = ["coding", "python", "markdown", "tutorial"]
date = "2023-12-01"
draft = false
+++

# Introduction

In this tutorial, we will guide you through creating a Python script that converts project directories into a structured format suitable for feeding code to AI/LLM (Large Language Models). This script will help you prepare your codebase by generating a structured file that includes the project structure and the contents of the files, making it easier for the AI/LLM to process and understand the code.

## Project Structure

Here's the structure of the project:

```
merge-to-markdown/
│
├── main.py
```

## Step 1: Setting Up the Script

First, let's set up the basic structure of the script. We'll initialize the necessary libraries and define the default extensions and output file.

```python
import argparse
from pathlib import Path
import tkinter as tk

# Updated to include more markdown-supported languages
DEFAULT_EXTENSIONS = [
    "ts", "py", "css", "scss", "html", "js", "jsx", "tsx", "yaml", "yml",
    "json", "md", "c", "cpp", "java", "rb", "go", "rs", "sh", "pl", "php",
    "swift", "kotlin", "r", "m", "h"
]
DEFAULT_OUTPUT_FILE = "merged.md"
```

## Step 2: Escaping Markdown Characters

We need to escape special Markdown characters to ensure they are displayed correctly in the generated Markdown file.

```python
def escape_markdown_characters(text):
    escaped_text = text.replace("\\", "\\\\").replace("_", "\\_").replace("#", "\\#").replace("`", "\\`")
    return escaped_text
```

## Step 3: Creating a Folder Structure Illustration

Next, we'll create a function to generate an illustration of the folder structure. This function will recursively traverse the directory and create a textual representation of the folder structure.

```python
def create_folder_structure_illustration(root_path, extensions, skip_folders):
    tree = ""
    for path in sorted(root_path.rglob('*')):
        if any(skip_folder in path.parts for skip_folder in skip_folders):
            continue
        if path.is_dir():
            depth = len(path.relative_to(root_path).parts)
            indent = '    ' * depth
            tree += f"{indent}{path.name}/\n"
        elif path.suffix[1:] in extensions:
            depth = len(path.relative_to(root_path).parts) - 1
            indent = '    ' * (depth + 1)
            tree += f"{indent}{path.name}\n"
    return tree
```

## Step 4: Merging Files

We'll create a function to merge the contents of the files into a single structured file. This function will read the contents of each file and add them to the structured file with the appropriate syntax highlighting.

```python
def merge_files(extensions, root_path, skip_folders):
    content = ""
    for ext in extensions:
        for file_path in root_path.rglob(f"*.{ext}"):
            if any(skip_folder in file_path.parts for skip_folder in skip_folders):
                continue
            try:
                file_path_str = str(file_path)
                escaped_file_path = escape_markdown_characters(file_path_str)
                content += f"\n## {escaped_file_path}\n"

                language_map = {
                    "py": "python",
                    "html": "html",
                    "css": "css",
                    "js": "javascript",
                    "jsx": "jsx",
                    "tsx": "tsx",
                    "ts": "typescript",
                    "scss": "scss",
                    "yaml": "yaml",
                    "yml": "yaml",
                    "json": "json",
                    "md": "markdown",
                    "c": "c",
                    "cpp": "cpp",
                    "java": "java",
                    "rb": "ruby",
                    "go": "go",
                    "rs": "rust",
                    "sh": "shell",
                    "pl": "perl",
                    "php": "php",
                    "swift": "swift",
                    "kotlin": "kotlin",
                    "r": "r",
                    "m": "objectivec",
                    "h": "c"
                }

                language = language_map.get(ext, "")
                if language:
                    content += f"```{language}\n"
                else:
                    content += "```\n"  # fallback to a default code block if extension is not in the map

                with file_path.open("r", encoding="utf-8") as f:
                    content += f.read()

                content += "\n```\n"
            except Exception as e:
                print(f"Error reading file {file_path}: {e}")
    return content
```

## Step 5: Saving to File

We'll create a function to save the merged content to a structured file.

```python
def save_to_file(content, output_path):
    try:
        with output_path.open("w", encoding="utf-8") as file:
            file.write(content)
    except Exception as e:
        print(f"Error writing to output file {output_path}: {e}")
```

## Step 6: Copying to Clipboard

We'll create a function to copy the merged content to the clipboard.

```python
def copy_to_clipboard(content):
    root = tk.Tk()
    root.withdraw()  # Hide the main window
    root.clipboard_clear()
    root.clipboard_append(content)
    root.update()  # Now it stays on the clipboard after the window is closed
    root.destroy()
```

## Step 7: Main Function

Finally, we'll create the main function to parse the command-line arguments and execute the script.

```python
def main():
    parser = argparse.ArgumentParser(description="Merge files with specified extensions.")
    parser.add_argument("-e", "--extensions", nargs="*", help="List of file extensions to merge.")
    parser.add_argument("-f", "--filename", help="Name of the output file (default: merged.md).")
    parser.add_argument("-s", "--skip-folders", nargs="*", help="List of folder names to skip.")
    args = parser.parse_args()

    extensions = args.extensions or DEFAULT_EXTENSIONS
    skip_folders = args.skip_folders or []
    root_path = Path.cwd()

    folder_structure = create_folder_structure_illustration(root_path, extensions, skip_folders)
    content = merge_files(extensions, root_path, skip_folders)

    full_content = f"## Project Structure\n```\n{folder_structure}```\n{content}"

    print("Merging:")
    if args.filename:
        output_path = Path(args.filename)
        save_to_file(full_content, output_path)
        print(f"Merged and saved to: {output_path}")
    else:
        copy_to_clipboard(full_content)
        print(folder_structure)
        print("Merged and copied to clipboard!")

if __name__ == "__main__":
    main()
```

## Conclusion

You've now created a Python script that converts project directories into a structured format suitable for feeding code to AI/LLM. This script will help you prepare your codebase by generating a structured file that includes the project structure and the contents of the files, making it easier for the AI/LLM to process and understand the code. You can further enhance the script by adding more features or improving the existing ones.

**update**

The script is now a command-line program published on pypy named 'code-merger'. 


### Resources

- The code for this tutorial [on my GitHub](https://github.com/raphael2692/code-merger/blob/main/merge/merge.py)