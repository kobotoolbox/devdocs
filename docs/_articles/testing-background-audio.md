---
title: Testing background-audio questions
---

When you need to test `background-audio` locally, the best way is to create fake submission instead of emulating Collect app.

Things to do:

1. Enable anonymous submissions for your project
2. Make a submission to your project with Enketo, and then get the XML for that submission from the API, e.g.
   ```
   http://kf.kobo.local:8999/api/v2/assets/arSQTmJnTXn5vMvDpiwypb/data/91.xml
   ```
3. Delete your submission :)
4. Edit the XML to change `<background-audio/>` to include a filename, e.g. `<background-audio>voiceover.ogg</background-audio>`
5. Save your XML file as `submission.xml`
6. Submit your XML—and include the audio file, why not? Make sure you send your request to `http://your-kc-host/YOUR-USERNAME/submission`. Example sending `voiceover.ogg`:
   ```
   curl -v -X POST -F xml_submission_file=@submission.xml -F voiceover.ogg=@voiceover.ogg http://kc.kobo.local:8999/aa/submission
   ```

Your `submission.xml` would look like:

```xml
<?xml version="1.0" encoding="utf-8"?>
<arSQTmJnTXn5vMvDpiwypb xmlns:jr="http://openrosa.org/javarosa" xmlns:orx="http://openrosa.org/xforms" id="arSQTmJnTXn5vMvDpiwypb" version="1 (2024-05-10 22:12:00)">
  <formhub>
    <uuid>0f705cfafaf54b4baa8f8f86a6606d1c</uuid>
  </formhub>
  <just_some_text>here's your text</just_some_text>
  <background-audio>voiceover.ogg</background-audio>
  <__version__>vSvhM59F2djU5x4ZWaQEeG</__version__>
  <meta>
    <instanceID>uuid:707ebc2e-ad46-49ba-9778-9aa8a78329a9</instanceID>
  </meta>
</arSQTmJnTXn5vMvDpiwypb>
```

You should see `<message>Successful submission.</message>` in the response from `curl`, and, of course, have a nice, playable background audio file in KPI now :)
