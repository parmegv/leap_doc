LEAP DOCUMENTATION
=================================

Files in the directory "docs" show up automatically at https://leap.se/docs when this repository is pushed.

Directory structure
---------------------------------

Every directory corresponds to a single web page. Within each directory, there are source files for different languages.

For example:

    docs/
      client/
        en.haml    -- used for https://leap.se/en/docs/client
        es.haml    -- used for https://leap.se/es/docs/client

If you don't care about translating a page, and it has no children, you can omit the directory:

    docs/
      client.haml  -- user for https://leap.se/*/docs/client

Menu
---------------------------------

A page does not show up in the navigation unless it appears in menu.txt.

The order in menu.txt determines the order in the navigation.


Localization
---------------------------------

The strings for titles and navigation menu are loaded from locales/*.yaml files. (alas, not yet in this repo, still in leap_website)

Supported syntax
---------------------------------

* .haml -- parsed as HAML file.
* .md -- parsed a Markdown.

