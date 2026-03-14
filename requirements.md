# Functional requirements

1. The script runs **start to finish as a single command** in a folder containing MP3 files.
2. It takes a directory of **multiple `.mp3` files** and produces **one `.m4b` audiobook**.
3. The output must include **one chapter per input MP3 file**.
4. Chapters must be created in the **correct file order**.
5. It must work with filenames that contain **spaces**, like `Fugue 001.mp3`.
6. It should support an optional **cover image** such as `Fugue.jpg`.
7. The final output should be a standard **M4B-compatible MP4/AAC file**.
8. The chapter titles should be based on the **input filenames**.
9. The script should be reusable for **other folders full of MP3 files**, not just this one example.
10. The script should require only **one invocation by the user**, not multiple manual steps.

# Performance requirements

1. The workflow should use **multiple CPU cores**.
2. It should use **6 cores/workers** as the safe default on your machine.
3. Since the job is CPU-bound, the expensive part should be **parallel AAC encoding**.
4. The final merge step should avoid re-encoding and instead use **fast concat/remux** where possible.

# Input assumptions

1. The folder contains a set of MP3 files that belong in one audiobook.
2. The files are already named in the desired order, for example zero-padded names like:

   * `Book 001.mp3`
   * `Book 002.mp3`
   * …
3. The folder may also contain a cover image file such as `*.jpg`.
4. The script should not depend on the book being named `Book`; it should work for other folders/books too.

# Output requirements

1. Produce a single `.m4b` file in the current folder.
2. The output filename should be derived automatically from the folder name or a script argument.
3. The output must preserve:

   * merged audio
   * chapters
   * optional cover art
4. The file should be playable in typical audiobook apps/players.

# Technical requirements

1. Use **ffmpeg** and **ffprobe**.
2. Generate chapter metadata in **FFMETADATA** format.
3. Determine chapter boundaries from the **actual encoded file durations**.
4. Encode each source MP3 to **AAC/M4A** first.
5. Do those per-file encodes in parallel, such as with `xargs -P 6` or equivalent.
6. Concatenate the intermediate `.m4a` files using FFmpeg’s **concat demuxer**.
7. Perform the final assembly with **`-c copy`** for the concat/remux stage.
8. Attach cover art as an **attached picture** when present.
9. Handle shell-safe paths and filenames robustly.

# Usability requirements

1. The script should be easy to run, such as:

   ```bash
   ./make_m4b.sh
   ```

   or

   ```bash
   ./make_m4b.sh /path/to/book-folder
   ```
2. It should not require hand-editing intermediate files.
3. It should clean up temporary files automatically.
4. It should fail clearly if:

   * no MP3 files are found
   * ffmpeg/ffprobe are missing
   * encoding fails
5. It should print a clear success message with the created output file.

# Nice-to-have requirements

1. Allow configurable settings:

   * number of jobs, default `6`
   * AAC bitrate, default `64k`
   * output filename
2. Optionally infer metadata like:

   * title
   * artist/author
   * album
3. Optionally accept a chosen cover image if more than one image exists.
4. Optionally support `.jpeg` and `.png` in addition to `.jpg`.
