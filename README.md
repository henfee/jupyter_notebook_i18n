# jupyter_notebook_i18n

Implementation Notes for Internationalization of Jupyter Notebook, Chinese Language

 According the Following guide, it works well.

![edit_UI](notebook_chinese_UI.png)

![help_UI](notebook_chinese_help.png)
 
 The simple way, copy this three files in notebook/i18n/zh_CN/LC_MESSAGES/, then start jupyter notebook:
 * notebook.mo
 * nbui.mo
 * nbjs.json

 I modified some translate words in Chinese language.


 ---

# [Implementation Notes for Internationalization of Jupyter Notebook](https://github.com/jupyter/notebook/blob/master/notebook/i18n/README.md)

The implementation of i18n features for jupyter notebook is still a work-in-progress:

- User interface strings are (mostly) handled
- Console messages are not handled (their usefulness in a translated environment is questionable)
- Tooling has to be refined

However…

## How the language is selected ?

1. `jupyter notebook` command reads the `LANG` environment variable at startup,
(`xx_XX` or just `xx` form, where `xx` is the language code you're wanting to
run in).

Hint: if running Windows, you can set it in PowerShell with `${Env:LANG} = "xx_XX"`.
      if running Ubuntu 14, you should set environment variable `LANGUAGE="xx_XX"`.

2. The preferred language for web pages in your browser settings (`xx`) is
   also used. At the moment, it has to be first in the list.

## Contributing and managing translations

### Requirements

- *pybabel* (could be installed `pip install babel`)
- *po2json* (could be installed with `npm install -g po2json`)

**All i18n-related commands are done from the related directory :**

    cd notebook/i18n/

### Message extraction

The translatable material for notebook is split into 3 `.pot` files, as follows:

- *notebook/i18n/notebook.pot* - Console and startup messages, basically anything that is
	produced by Python code.
- *notebook/i18n/nbui.pot* - User interface strings, as extracted from the Jinja2 templates
	in *notebook/templates/\*.html*
- *noteook/i18n/nbjs.pot* - JavaScript strings and dialogs, which contain much of the visible
	user interface for Jupyter notebook.

To extract the messages from the source code whenever new material is added, use the
`pybabel` command:

```shell
pybabel extract -F babel_notebook.cfg -o notebook.pot --no-wrap --project Jupyter .
pybabel extract -F babel_nbui.cfg -o nbui.pot --no-wrap --project Jupyter .
pybabel extract -F babel_nbjs.cfg -o nbjs.pot --no-wrap --project Jupyter .
```

After this is complete you have 3 `.pot` files that you can give to a translator for your favorite language.

### Messages compilation

After the source material has been translated, you should have 3 `.po` files with the same base names
as the `.pot` files above.  Put them in `notebook/i18n/${LANG}/LC_MESSAGES`, where `${LANG}` is the language
code for your desired language ( i.e. German = "de", Japanese = "ja", etc. ).

*notebook.po* and *nbui.po* need to be converted from `.po` to `.mo` format for
use at runtime.

```shell
pybabel compile -D notebook -f -l ${LANG} -i ${LANG}/LC_MESSAGES/notebook.po -o ${LANG}/LC_MESSAGES/notebook.mo
pybabel compile -D nbui -f -l ${LANG} -i ${LANG}/LC_MESSAGES/nbui.po -o ${LANG}/LC_MESSAGES/nbui.mo
```

*nbjs.po* needs to be converted to JSON for use within the JavaScript code, with  *po2json*, as follows:

    po2json -p -F -f jed1.x -d nbjs ${LANG}/LC_MESSAGES/nbjs.po ${LANG}/LC_MESSAGES/nbjs.json

When new languages get added, their language codes should be added to *notebook/i18n/nbjs.json*
under the `supported_languages` element.

### Tips for Jupyter developers

The biggest "mistake" I found while doing i18n enablement was the habit of constructing UI messages
from English "piece parts".  For example, code like:

```javascript
var msg = "Enter a new " + type + "name:"
```

where `type` is either "file", "directory", or "notebook"....

is problematic when doing translations, because the surrounding text may need to vary
depending on the inserted word. In this case, you need to switch it and use complete phrases,
as follows:

```javascript
var rename_msg = function (type) {
    switch(type) {
        case 'file': return _("Enter a new file name:");
        case 'directory': return _("Enter a new directory name:");
        case 'notebook': return _("Enter a new notebook name:");
        default: return _("Enter a new name:");
    }
}
```

Also you need to remember that adding an "s" or "es" to an English word to
create the plural form doesn't translate well.  Some languages have as many as 5 or 6 different
plural forms for differing numbers, so using an API such as ngettext() is necessary in order
to handle these cases properly.

### Known issues and future evolutions

1. Right now there are two different places where the desired language is set.  At startup time, the Jupyter console's messages pay attention to the setting of the `${LANG}` environment variable
as set in the shell at startup time.  Unfortunately, this is also the time where the Jinja2
environment is set up, which means that the template stuff will always come from this setting.
We really want to be paying attention to the browser's settings for the stuff that happens in the
browser, so we need to be able to retrieve this information after the browser is started and somehow
communicate this back to Jinja2.  So far, I haven't yet figured out how to do this, which means that if the ${LANG} at startup doesn't match the browser's settings, you could potentially get a mix
of languages in the UI ( never a good thing ).

2. We will need to decide if console messages should be translatable, and enable them if desired.
3. The keyboard shorcut editor was implemented after the i18n work was completed, so that portion
does not have translation support at this time.
4. Babel's documentation has instructions on how to integrate messages extraction
into your *setup.py* so that eventually we can just do:

    ./setup.py extract_messages

I hope to get this working at some point in the near future.
5. The conversions from `.po` to `.mo` probably can and should be done using `setup.py install`.


Any questions or comments please let me know @JCEmmons on github (emmo@us.ibm.com)
