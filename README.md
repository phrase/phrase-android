# Phrase Over the Air SDK for Android

Publish your translations faster and simpler than ever before. Stop waiting for the next deployment and start publishing all your translations in real-time directly in Phrase.

## Instructions

Head over to the Phrase Help Center to learn about this feature and how to use it in your apps: https://support.phrase.com/hc/en-us/articles/5804059067804-Over-the-Air-Strings-

## Known limitations

- Inflated menus and preferences are not supported yet
- Some libraries that do not support automatic context unwrapping can cause issues. For example `Hilt` requires to disable automatic context wrapping in Compose components https://github.com/phrase/phrase-android/releases/tag/3.5.0
