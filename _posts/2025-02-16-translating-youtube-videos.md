---
layout: post
title: "Transcibing Youtube Videos to English using Whisper and Deep-translator"
date: "2025-02-16 18:00:00 +0000"
categories: til
tags: [python, whisper, translation, yt_dlp, dutch, year of health]
excerpt: " "
---

This year is, for me, the Year of Health, this encompasses many things, the usual like getting fitter and stronger, maintaining relationships etc., as well as less obviously "healthy" things like learning. One of my main goals is to finally learn Dutch to a decent level. I'm aiming for [CEFR](https://en.wikipedia.org/wiki/Common_European_Framework_of_Reference_for_Languages) level B1 by the middle of the year, and B2/C1 by the end of the year. One of the hardest aspects for beginner language learners is to find interesting content at their level. I couldn't find anything with translations at my level so I decided I'd make my own. The idea is to translate YouTube videos into Dutch and English transcripts, so I can follow along with a Dutch-English script. This turns out to be surprisingly straightforward using Python, Whisper, and Deep-Translator.

## Overview:

**Step 1. Download a YouTube video and convert to MP3.**

- Use [yt_dlp](https://github.com/yt-dlp/yt-dlp) to download the vidoes with the best available quality and convert it to MP£ using [ffmpeg](https://www.ffmpeg.org).

**Step 2. Transcribe the MP3 to Dutch text\***

- Use OpenAI's open-source [whisper](https://github.com/openai/whisper) model which can be run locally. This model can do speech recognition in many languages and can be installed with: `pip install git+https://github.com/openai/whisper.git`.

**Step 3. Translate the Dutch text to English**

- Use [deep-translator](https://github.com/nidhaloff/deep-translator), specifically `GoogleTranslator`. `GoogleTranslator` sends text to Google's mobile translator endpoint, which has a limit of 5000 characters, so if the Dutch text is too long, it must be split into chunks.

## Core code:

```python
import os
import glob
import yt_dlp
import whisper
import re
import pandas as pd
from deep_translator import GoogleTranslator  # Make sure to import this

CONFIG = {
    'URL': "https://www.youtube.com/watch?v=cSZQLi5U0FA",
    'FILENAME_BASE': 'audio',
    'SOURCE_LANG': 'nl',
    'TARGET_LANG': 'en',
    'MODEL_SIZE': 'small'  # Try 'tiny' if memory or speed is an issue
}

def download_audio(url, filename_base):
    """Download audio from a YouTube video using yt-dlp and convert to MP3."""
    ydl_options = {
        'format': 'bestaudio/best',
        'outtmpl': filename_base,
        'postprocessors': [{
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'mp3',
            'preferredquality': '128'
        }],
        'quiet': True,
        'no_warnings': True,
    }

    print(f"Downloading audio from: {url}")
    with yt_dlp.YoutubeDL(ydl_options) as ydl:
        info = ydl.extract_info(url, download=False)
        video_title = info.get('title', 'Unknown Title')
        print(f'Video title: {video_title}')
        ydl.download([url])
    print("Download complete!")

download_audio(CONFIG['URL'], CONFIG['FILENAME_BASE'])

def transcribe_audio(filename_base, language, model_size):
    """Transcribe an audio file using Whisper."""
    # If necessary, supply extra arguments, e.g. 'task="transcribe"'
    model = whisper.load_model(model_size)
    audio_filename = f"{filename_base}.mp3"
    result = model.transcribe(audio_filename, language=language)
    return result["text"]

dutch_text = transcribe_audio(CONFIG['FILENAME_BASE'], CONFIG['SOURCE_LANG'], CONFIG['MODEL_SIZE'])

def split_text(text, max_length=5000):
    """Split text into chunks of max_length, respecting sentence boundaries."""
    if len(text) <= max_length:
        return [text]

    chunks = []
    sentences = re.split(r'(?<=[.!?])\s+', text)
    current_chunk = ""

    for sentence in sentences:
        if len(current_chunk) + len(sentence) <= max_length:
            current_chunk += sentence + " "
        else:
            chunks.append(current_chunk.strip())
            current_chunk = sentence + " "
    if current_chunk.strip():
        chunks.append(current_chunk.strip())

    print(f"Text split into {len(chunks)} chunks for translation.")
    return chunks

dutch_text_chunks = split_text(dutch_text)

def translate_chunks(chunks, source_lang, target_lang):
    """Translate a list of text chunks using GoogleTranslator from Deep-Translator."""
    translator = GoogleTranslator(source=source_lang, target=target_lang)
    return [translator.translate(chunk) for chunk in chunks]

english_text_chunks = translate_chunks(dutch_text_chunks, CONFIG['SOURCE_LANG'], CONFIG['TARGET_LANG'])
english_text = "\n\n***\n\n".join(english_text_chunks)

def convert_to_df(source_text, translated_text):
    """Convert Dutch and English texts into a pandas DataFrame."""
    source_sentences = re.split(r'(?<=[.!?])\s+', source_text.strip())
    translated_sentences = re.split(r'(?<=[.!?])\s+', translated_text.strip())

    # Pad out lists so they have the same length
    max_len = max(len(source_sentences), len(translated_sentences))
    source_sentences += [""] * (max_len - len(source_sentences))
    translated_sentences += [""] * (max_len - len(translated_sentences))

    df = pd.DataFrame({
        "Dutch (Original)": source_sentences,
        "English (Translation)": translated_sentences
    })
    return df

df = convert_to_df(dutch_text, english_text)
df.to_csv("dutch_english_translation.csv", index=False)
```

## Notes

- **Sentence Alignment Bug**: Because we are splitting on `[.!?]`, variations in punctuation between Dutch and English spelling can lead to misaligned rows (e.g., "meneer" -> "Mr."). This shifts sentence counts by an extra period. I could mitigate this by manually cleaning punctuation, or using more advanced segmentation libraries (e.g. [spaCy](https://github.com/explosion/spaCy) or [NLTK](https://www.nltk.org)) for both languages. For the current use case it works so I can't be bothered fixing this bug at the moment.
- **`ffmpeg`**: Currently `yt_dlp` uses `ffmpeg` under the hood to convert the video to MP3. This can be `ffmpeg` can be installed on mac using `brew install ffmpeg`, and the version verified using `ffmpeg -version`.
- **Deep-Translator**: Heavy use of this could be rate-limited from Google, but I haven't had a problem translating hours of text (~40 x 5,000 character chunks concurrently).

Overall, as long as you're willing to do a bit of post-processing (which is not a bad excercise in itself when you are learning a e), this setup works well enough to get Dutch-English transcripts. I've also written a more feature-complete script, with command-line options at my github repo [seamusbarnes/youtube_transcribe](https://github.com/seamusbarnes/youtube_transcribe).
