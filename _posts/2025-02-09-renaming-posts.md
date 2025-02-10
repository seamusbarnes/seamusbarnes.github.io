---
layout: post
title: "Renaming Blogposts Using ruamel.yaml"
date: "2025-02-09 19:00:00 +0000"
categories: til
tags: [python, blog, yaml]
# excerpt: "testest"
---

I wanted to rename all my TIL blog posts from \[Title\] -> \[TIL: Title\], because this blog might be evolving from a TIL blog to an _everything_ blog (with link posts, personal posts, reflections, etc.). So I did what I usually do when I want to automate something in a way I've never done before, I asked ChatGPT. In the process I learned about the `ruamel.yaml` and `io` packages...

```python
import os # for interacting with the filesystem
import re # for regex operations
from io import StringIO # for using an in-memory file-like object
from ruamel.yaml import YAML # YAML parser that preserves formatting

# directory where your Jekyll posts are stored
POSTS_DIR = "_posts"

# initialize YAML parser with round-trip support (preserves formatting)
yaml = YAML()
yaml.preserve_quotes = True  # ensures quotes in strings are not altered
yaml.default_flow_style = False # keeps YAML formatting readable (not compact inline style)

# Process each Markdown file in _posts/
for filename in os.listdir(POSTS_DIR):
    if filename.endswith(".md"): # ensures we only process Markdown files
        file_path = os.path.join(POSTS_DIR, filename)

        # open the file in read mode to get its content
        with open(file_path, "r", encoding="utf-8") as f:
            content = f.read()

        # use regex to extract YAML front matter and post content
        match = re.match(r"^---\n(.*?)\n---\n(.*)$", content, re.DOTALL)
        if not match: # if the regex doesn't match, it means there's no valid front matter
            print(f"Skipping {filename}: No valid front matter found.")
            continue # skip this file and move to the next one

        front_matter, body = match.groups() # extract the YAML front matter and post body

        # load the extracted YAML front matter into a Python dictionary
        yaml_data = yaml.load(front_matter)

        # retrieve the current title, if present
        title = yaml_data.get("title", "")

        if title and not title.startswith("TIL: "): # check if title needs updating
            yaml_data["title"] = f"TIL: {title}" # prepend "TIL: " to the title

            # use StringIO as a temporary in-memory file to store the updated YAML front matter
            stream = StringIO()
            yaml.dump(yaml_data, stream) # dump updated YAML data into the stream
            new_front_matter = stream.getvalue() # retrieve the updated YAML as a string

            # Reassemble the complete file with updated front matter
            new_content = f"---\n{new_front_matter}---\n{body}"

            # potential pitfall: risk of file corruption
            # opening the file in write mode ("w") would erase its content before writing the new content
            # instead, you could us "r+" (read/write mode) to prevent accidental data loss in case of errors
            # however, "r+" still has risks, so a safer option is writing to a temporary file first
            with open(file_path, "w", encoding="utf-8") as f:
                f.write(new_content) # write the updated content back to the file

            print(f"Updated: {filename}") # log the update
        else:
            print(f"Skipped (already prefixed): {filename}") # skip files already updated
```

YAML (Yet Another Markup Language) is a human-readable data format used for config files and front matter in Jekyll blogs. `ruamel.yaml` reads and writes YAML files or text while preserving formatting, indentation and comments. `StringIO` provides an in-memory file-like object . `yaml.dump(yaml_data, stream)` converts the `yaml_data` dictionary properly formatted YAML text and writes it to `stream`, which is an in-memory file-like object probided by StringIO. This avoids unnecessary disk writes and ensures that the YAML remains correctly formatted before modifying the original file.

There are lots of subtle ways this can go wrong. For example, at some point I was editing the code and adding `break` statements at different points. At one point I realised I had wiped the contents from one of the blog posts, "because I had opened it in write mode, which erased the file's contents before any new content was written:

```python
with open (file_path, "w", encoding="utf-8") as f:
    break # deletes content of file!
    f.write(new_content)
```

Luckily I could revert this change with the git command: `git restore <file>`.

This was a fun little exercise and it's just another example (along with using python for generating Anki cards in a specific format from a dataset) of how python can be used for automating file manipulation.
