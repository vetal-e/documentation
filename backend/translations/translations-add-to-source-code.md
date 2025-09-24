<a id="dev-translation-add-to-source-code"></a>

# Add Translations to Source Code

Out-of-the-box, only base English translations (`en` language code) are loaded from the translation files.
As described in <a href="https://symfony.com/doc/current/translation.html#translation-resource-file-names-and-locations" target="_blank">translation files</a>, these files are located in the Resources/translations directory of any bundle and the translations directory of the application, e.g., Resources/translations/messages.en.yml, Resources/translations/validators.en.yml, etc.

These translations are compiled in the [Symfony translation catalogs](translations.md#dev-translation-symfony-translator), which are sets of PHP files. As a result, these files are cached by <a href="http://php.net/manual/en/intro.opcache.php" target="_blank">OPcache</a>, so getting these translations is fairly quick.

You can download translations for other languages from Crowdin or added manually in the Back-Office. These translations are loaded from the database to the application cache, and getting them is slower than from the Symfony translation catalogs. To minimize performance issues, consider adding them to the source code, as described below:

1. Use oro:translation:dump-files to dump translations to the translations directory of the application.
2. When you have updated the existing translations, use a file comparison tool of your choice to verify the dumped files.
3. Push the files to the version control system (git).

To rebuild Symfony translation catalogs with new translations and remove them from the database, use the `oro:platform:update` command.

Please take into account that languages loaded from the dumped files will be disabled from management via Crowdin in the UI and the data should be managed by a developer.

<!-- Frontend -->
