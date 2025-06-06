# Essências by Gabriel Dall'Alba

This repository contains the source code for the personal website of Gabriel Dall'Alba, hosting poetry and philosophical reflections. The website is built using [Hugo](https://gohugo.io/) with a modified version of the [PaperMod theme](https://github.com/adityatelange/hugo-PaperMod). Built upon a fabulous template by [Pascal Michaillat](https://pascalmichaillat.org/b/).

## Website Structure

The website is multilingual (Portuguese and English, soon mandarin and japanese) and contains the following main sections:

- **Poetry/Poesia**: Collection of poems organized by themes and collections
- **Philosophy/Filosofia**: Philosophical reflections and essays
- **About/Sobre**: Information about the author

## Local Development - for developer

To work on this website locally from a different computer, for any erason:

1. Install [Hugo](https://gohugo.io/installation/) (v0.147.2 or later) (Remember to install Go dependency, used scoop to install HUGO)
2. Clone this repository
3. Navigate to the repository directory
4. Run the Hugo development server:

```bash
hugo server
```

The website will be available at http://localhost:1313/essencias/, with live reloading enabled as you edit files.

## Project Structure

`content`: All website content organized by language and section

`en/`: English content

`poesias/`: Portuguese poetry content

`filosofia/`: Portuguese philosophy content

`sobre/`: Portuguese about page

`layouts`: Custom layout templates

`PaperMod`: The base theme

`static`: Static files (images, PDFs, etc.)

`archetypes`: Templates for new content

`config.yml`: Site configuration

`workflows`: GitHub Actions workflow for deployment

## Deployment

The website is automatically deployed to GitHub Pages using GitHub Actions when changes are pushed to the main branch. The deployment workflow is defined in `hugo.yml`.

## A case of switching languages

I wanted to expand upon the existing template and implement different language pages that you can swiftly change. Take for example the [first poem of the website](https://gdalba.github.io/essencias/poesias/aopassodalargada/). By default, the website is in Portuguese-Brazil. My current implementation allows the reader to swiftly press the `en` button on the upper left and change to English. Notice how the URL changes from `/essencias/poesias/aopassodalargada/` to `/essencias/en/poetry/aopassodalargada/`. To achieve this, I developed a language switch:

```html
<ul class="lang-switch">
    <li>
        <a href="{{ .RelPermalink 
                    | replaceRE `^/essencias/en/` `/essencias/` 
                    | replaceRE `/poetry/` `/poesias/` 
                    | replaceRE `/philosophy/` `/filosofia/` 
                    | replaceRE `/about/` `/sobre/` 
                    | replaceRE `/themes/` `/temas/` }}" 
           title="Português" 
           aria-label="Português">Pt</a>
    </li>
    <li>
        <a href="{{ .RelPermalink 
                    | replaceRE `^/essencias/(poesias|filosofia|sobre|temas)` `/essencias/en/${1}` 
                    | replaceRE `/poesias/` `/poetry/` 
                    | replaceRE `/filosofia/` `/philosophy/` 
                    | replaceRE `/sobre/` `/about/` 
                    | replaceRE `/temas/` `/themes/` }}" 
           title="English" 
           aria-label="English">En</a>
    </li>
</ul>

```

### A two-step approach does as follows:

#### From a Portuguese link:

- Step 1: Match and remove the `/en/` segment that comes after `/essencias/`

It assumes all English pages live under `/essencias/en/`, this is the "namespace" for English content.

- Step 2: Translate known categories (like `/poetry/`) into their Portuguese equivalents

`/poetry/` → `/poesias/`

`/about/` → `/sobre/`

This allows clean URL mirroring between languages.

#### From an English Link:

Step 1: Insert `/en/` after `/essencias/`, but only if the page is one of those known Portuguese categories

The regular expression (`poesias` | `filosofia` | `...`) captures the category, `${1}` puts it back in the correct place, but now under `/en/`

Step 2: Translate the category to English:

`/poesias/` → `/poetry/`

`/filosofia/` → `/philosophy/`

#### The foolish limitation

Obviously, pairwise at n = 2, this works well, but as n increases, the `replaceRE` approach will grow as each source → target pair must be listed; equally so, let:

- **L** = number of languages  
- **S** = number of categories (e.g., `poetry`, `about`, `themes`, etc.)

Each category requires mappings from **every language to every other language**, which is:

**Mappings per categories** = `L * (L - 1)`  
**Total mappings across all categories** = `S * L * (L - 1)`

---

## 📊 Examples

### 🔹 2 Languages (`L = 2`), 5 Categories (`S = 5`):

`5 * 2 * (2 - 1) = 10` mappings

### 🔹 3 Languages (`L = 3`), 5 Categories:

`5 * 3 * (3 - 1) = 30` mappings

### 🔹 5 Languages (`L = 5`), 7 Categories:

`7 * 5 * (5 - 1) = 140` mappings

A solution must be found, because I don't want to use Hugo's native translation. (Update: Currently, I do use the i18n approach, but implemented in union with my custom approach, this gives full translation to pages while staying in the same page when changing locale, it's quite nice now).

```html
                <ul class="lang-switch">
                    <li>
                        <a href="{{ .RelPermalink 
                                | replaceRE `^/essencias/en/` `/essencias/`
                                | replaceRE `/poetry/` `/poesias/` 
                                | replaceRE `/philosophy/` `/filosofia/` 
                                | replaceRE `/about/` `/sobre/` 
                                | replaceRE `/themes/` `/temas/` }}" 
                        title="{{ i18n "pt" }}" 
                        aria-label="{{ i18n "pt" }}">{{ i18n "pt" }}</a>
                    </li>
                    <li>
                        {{- $is_english_page := in .RelPermalink "/en/" }}
                        {{- if $is_english_page }}
                            <a href="#" class="active" 
                            title="{{ i18n "en" }}"  
                            aria-label="{{ i18n "en" }}">{{ i18n "en" }}</a>
                        {{- else }}
                            <a href="{{ .RelPermalink 
                                    | replaceRE `^/essencias/` `/essencias/en/` 
                                    | replaceRE `/poesias/` `/poetry/` 
                                    | replaceRE `/filosofia/` `/philosophy/` 
                                    | replaceRE `/sobre/` `/about/` 
                                    | replaceRE `/temas/` `/themes/` }}" 
                            title="{{ i18n "en" }}"  
                            aria-label="{{ i18n "en" }}">{{ i18n "en" }}</a>
                        {{- end }}
                    </li>
                </ul>
```

## How to use i18n

A folder called `i18n` contain what I perceive to be small dictionaries of translation, in there you will find `<language>.yml`, and it will call for the appropriate translations to take place. A key change is the addition of the tag `ContentDir` to the `config.yml` file, this way, Hugo knows how to call the folder with content of that specific language. I don't know exactly why but this fixed i18n implementation for me.

For new languages, create the `<language>.yml` inside ì18n` folder, and add the key IDs and translations within. Make sure to add the language and specifics to `config.yml`.

## License

The website code is licensed under the MIT License. Content is copyright © Gabriel Dall'Alba.

## Acknowledgments

[Hugo](https://gohugo.io/) for the static site generator.
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) for the base theme.
[Template by Pascal Michaillat](https://pascalmichaillat.org/b/) for baseline and additional theme customizations.
[Sara Zhang](https://saraz9.github.io/) for inspiration.
