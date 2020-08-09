
The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Scrape the results
==========================

```sh
bundle exec ruby scraper.rb https://en.wikipedia.org/wiki/2018_West_Tyrone_by-election | tee wikipedia.csv
```

This required changing the text of the h2 before the table: this one
uses "Candidates and result" rather thn "Result"

Step 2: Generate possible missing IDs
=====================================

```sh
xsv search -v -s id 'Q' wikipedia.csv | xsv select name | tail +2 |
  sed -e 's/^/"/' -e 's/$/"@en/' | paste -s - |
  xargs -0 wd sparql find-candidates.js |
  jq -r '.[] | [.name, .item.value, .election.label, .constituency.label, .party.label] | @csv' |
  tee candidates.csv
```

The one found looks good, but there's no match for Chris Smyth (UUP). I
can't find any Wikidata item for him either, so I think it's OK to
create one.

Step 3: Combine Those
=====================

```sh
xsv join -n --left 2 wikipedia.csv 1 candidates.csv | xsv select '7,1-5' | sed $'1i\\\nfoundid' > combo.csv
```

Step 4: Generate QuickStatements commands
=========================================

Tweak config variables in `generate-qs.rb`, and then:

```sh
bundle exec ruby generate-qs.rb | tee commands.qs
```

Then sent to QuickStatements as https://editgroups.toolforge.org/b/QSv2T/1596967911851/
