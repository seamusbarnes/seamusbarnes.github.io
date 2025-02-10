---
layout: post
title: "Using Python to Automatically Generate Anki Cloze Deletion Cards"
date: 2025-01-10 15:30:00 +0000
categories: til
tags: [anki, python]
---

You can use python to generate a `.txt` file from a `.csv` file to auto-generate many Anki cards with the same format, which you can then import simply into Anki. I used the following procedure to Ankify approximately 1000 Dutch vocabulary words.

I found a list of approx. 1000 English - Dutch words taken from Duolingo already compiled by Reddit user xBroleh [here](https://www.reddit.com/r/learndutch/comments/vsw8c3/ive_compiled_an_excel_spreadsheet_with_every/). The list is in google sheets so I exported as `.csv`. I then ran a simple Python code which read the `.csv` file and created a .txt file with each line having the format:

```text
{% raw %}
English: {{c1::[ENGLISH WORD]}}<br><br>Dutch: {{c2::{[DUTCH WORD]}}
{% endraw %}
```

When imported into Anki each line is treated as an individual card which looks like this:

<div style="text-align: center;">
  <p style="margin: 0; text-align: center; font-weight: bold">
    Front
  </p>
  <img src="{{ site.baseurl }}/assets/posts/anki-card-dutch-front.png" style="width: 75%;">
</div>

<div style="text-align: center;">
  <p style="margin: 0; text-align: center; font-weight: bold">
    Back
  </p>
  <img src="{{ site.baseurl }}/assets/posts/anki-card-dutch-back.png" style="width: 75%;">
</div>

_Python Code_

```python
import csv

input_csv = 'duolingo_dutch.csv'    # Name of input CSV file
output_txt = 'anki_cards.txt'       # Name of output file

# Open the input CSV file and output text file
with open(input_csv, 'r', encoding='utf-8') as f_in, open(output_txt, 'w', encoding='utf-8') as f_out:
    # Create a CSV reader object to read the input file
    reader = csv.reader(f_in, delimiter=',')

    # Iterate through each row in the CSV file
    for row in reader:
        # Each row is a list of column values. For example: ["de man", "man"]

        # Skip rows with fewer than 2 columns, as they don't contain useful
        if len(row) < 2:
            continue

        # Extract and clean the English, Dutch words from the row
        dutch = row[0].strip()
        english = row[1].strip()

        # Skip rows that are headers or irrelevant sections:
        # - Rows where 'Dutch' or 'English' appear as content
        # - Empty rows or rows starting with specific markers like '***'
        if dutch in ('Dutch', '') or english in ('English', ''):
            continue
        if dutch.startswith('***'):  # Skip lines that look like section dividers or irrelevant data
            continue

        # Format the data for Anki Cloze deletion cards:
        # - Add `<br>` tags for line breaks to ensure proper formatting in Anki
        {% raw %}line = f"English: {{{{c1::{english}}}}}<br><br>Dutch: {{{{c2::{dutch}}}}}\n"{% endraw %}

        # Write the formatted line to the output file
        f_out.write(line)
```

This generates a `.txt` fle which can be imprted into Anki. It is important to correctly set the _Field separator_ option to _Tab_, and the _Note Type_ to _Cloze_ so Anki's import tool recognises how the the field should be generated from each line, and that the {% raw %}{{c1::WORD}}{% endraw %} is regognised as a cloze deletion element.

<div style="text-align: center;">
  <!-- <p style="margin: 0; text-align: center; font-weight: bold">
  HEADER TEXT
  </p> -->
  <img src="{{ site.baseurl }}/assets/posts/anki-import-dutch-cards.png" style="width: 75%;">
</div>

This way of generating and formatting cards seems very powerful. The posibilities for more complex templates using HTML are endless. In this case I only used `<br>` linebreaks to separate the English from Dutch text, but you could do so much more if you needed to.
