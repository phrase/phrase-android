# Phrase Over-the-Air Migration Guide

### Update to v3.11.x+
Version 3.11.0 starts the process of removing the Android Context wrapping by Phrase.
Wrapping the Android Context can lead to severe issues with other libraries. E.g. App crashes can occur in combination of WebViews.

#### Suggested Actions:
- Replace `super.attachBaseContext(Phrase.wrapContext(base))` with `super.attachBaseContext(base)` in your Application class.
- Replace `Phrase.getDelegate(super.getDelegate())` with `Phrase.getDelegate(super.getDelegate(), wrapContext = false)` in your Activities.
- Replace the Android resource calls with the Phrase pendants:

  |Android Resource Call|Phrase Pendant|
  |-------------|--------------|
  |`Context.getString()`|`Context.getPhraseString()`|
  |`Context.getText()`|`Context.getPhraseText()`|
  |`Context.getStringArray()`|`Context.getPhraseStringArray()`|
  |`Context.getTextArray()`|`Context.getPhraseTextArray()`|
  |`Context.getQuantityString()`|`Context.getPhraseQuantityString()`|
  |`Context.getQuantityText()`|`Context.getPhraseQuantityText()`|
  |`Resources.getString()`|`Resources.getPhraseString()`|
  |`Resources.getText()`|`Resources.getPhraseText()`|
  |`Resources.getStringArray()`|`Resources.getPhraseStringArray()`|
  |`Resources.getTextArray()`|`Resources.getPhraseTextArray()`|
  |`Resources.getQuantityString()`|`Resources.getPhraseQuantityString()`|
  |`Resources.getQuantityText()`|`Resources.getPhraseQuantityText()`|
  |`TypedArray.getString()`|`TypedArray.getStringWithPhrase()`|
  |`TypedArray.getText()`|`TypedArray.getTextWithPhrase()`|
- Remove all `Phrase {}` wrapper composable calls if you used Phrase in Compose before.
- Use the Phrase calls to get translations in Compose:

  |Phrase OTA Composables|
  |-------|
  |`phraseString()`|
  |`phraseText()`|
  |`phraseStringArray()`|
  |`phraseTextArray()`|
  |`phraseQuantityString()`|
  |`phraseQuantityText()`|