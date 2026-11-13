# Date Format Pattern Translator

Paste a date format pattern and see what it prints under UTS 35, date-fns, moment, POSIX strftime and .NET for a date and region you pick, then get the equivalent pattern in each, including the late-December week-year trap where `YYYY` prints next year.

**Live demo:** https://0xelitesystem.github.io/date-format-pattern-translator/

## Use

Open the page, type a pattern, type a date. Two views update as you type.

**View A, the same string in every dialect.** The literal text you pasted is tokenized by five formatters side by side:

- UTS 35 (the Unicode pattern standard), with week rules from CLDR 48.2 for the region code you pick
- date-fns 4.x, with week options copied from any date-fns locale and both opt-in flags
- moment 2.x, en names, with any valid `dow` / `doy` week rule
- strftime, as POSIX.1-2024 only, glibc, or MSVC
- .NET custom format strings, InvariantCulture

Every token shows what it means in that dialect and what it printed, so `YYYY-MM-DD` reads as week-year, month, day-of-year in UTS 35, throws in date-fns, prints the plain date in moment, and echoes `YYYY-12-DD` in .NET.

**View B, the same meaning in each dialect.** Pick which dialect the pattern is written in. The page parses it into fields and writes each field in every other dialect. A field with no equivalent is named in the output, never dropped. Each translation is then rendered against the source by this page's engines on every day from 2020-01-01 to 2030-12-31, once at noon and once with the time of day stepped through all 24 hours, and the match count is shown. A translation that is only equal under matching week rules says so, and the count shows when it is not.

**Trap dates.** For any year, the page computes every day whose week-based year is not its calendar year, under the UTS 35 region rule, the date-fns options, the moment rule and ISO 8601.

**Self test.** It runs on load in a collapsed panel and on demand: measured fixtures for each dialect, three independent oracles for the week arithmetic, a full round trip of every core mapping row, and a positive control that must fail and is shown as detected.

Real situations this is for:

1. **A report says 2027 on December 28, 2026.** Paste the pattern. If it holds `YYYY`, View A shows the week-based year, and the trap-dates table lists every day that will do it for your region.
2. **Moving off moment.** Read your moment pattern as moment and copy the date-fns line. `DD` becomes `dd`, `Do` becomes `do`, and `gggg` is flagged as week-rule dependent with the opt-in flag it needs. A request for this conversion has been open on the date-fns tracker since 2022-03-18 ([issue #2991](https://github.com/date-fns/date-fns/issues/2991)).
3. **Matching a C or Python log line to a .NET or JavaScript one.** Read the strftime pattern as strftime and copy the other lines. Where POSIX has no unpadded day or hour, the page says so and shows the glibc (`%-d`) and MSVC (`%#d`) forms separately.
4. **Week numbers across offices.** Switch the region between US, DE, PT and AU. PT starts weeks on Sunday with minDays 4, so it is not ISO 8601 even though its minDays matches.
5. **Code review.** Paste a pattern from a pull request and read the date-fns card: exactly `D`, `DD`, `YY` and `YYYY` throw RangeError unless the opt-in flag is set, while other runs of Y or D only warn.

## A difference worth knowing

date-fns reads week rules from its own locale files, and they do not always match CLDR. In date-fns 4.4.0, `enAU` and `enIN` both set `weekStartsOn: 1, firstWeekContainsDate: 4`. CLDR 48.2 has AU as Monday first with minDays 1, and IN as Sunday first with minDays 1. So `YYYY` on 2021-01-01 prints 2020 in date-fns with `enAU` and 2021 in ICU with `en_AU`, and `YYYY` on 2026-12-27 prints 2026 in date-fns with `enIN` and 2027 in ICU with `en_IN` (measured 2026-10-01 with date-fns 4.4.0 from npm and ICU 77.1 through PHP intl). Both follow their own published data. The page computes the same results from the embedded tables; set the region to AU or IN and the date-fns options to `enAU` or `enIN` to see it.

## Why this exists

Date pattern letters look alike across libraries and mean different things. The same five characters can print a week-based year, a calendar year, or themselves. This is a single HTML file that shows all five readings at once and checks its own translations by computation rather than by a lookup table. No tracking, no sign-up, no dependencies, MIT licensed.

## Privacy

Everything runs in your browser. Nothing you type is uploaded or stored on a server; the page makes no network requests, and its Content-Security-Policy forbids them. The only thing kept is your light or dark theme choice, in your own browser's local storage.

## Run locally

```bash
git clone https://github.com/0xelitesystem/date-format-pattern-translator
cd date-format-pattern-translator
# open index.html in a browser, or serve it:
python -m http.server 8000
```

## Build

There is no build. It is one `index.html` with inline CSS and JavaScript.

## What is modeled, and the limits

- **UTS 35**: field meanings and valid lengths from the [Part 4 Date Field Symbol Table](https://www.unicode.org/reports/tr35/tr35-dates.html). A valid letter at an invalid length prints U+FFFD and an undefined letter is an error, as [Part 1, Handling Invalid Patterns](https://www.unicode.org/reports/tr35/tr35.html) recommends. That is spec behavior, labeled as such; libraries differ. Week data is generated from CLDR [release-48-2 supplementalData.xml](https://github.com/unicode-org/cldr/blob/release-48-2/common/supplemental/supplementalData.xml), the latest CLDR release when this was built (published 2026-03-17). Names are CLDR English for every region. Dates use the proleptic Gregorian calendar, years 1000 to 9999.
- **CLDR 47 to 48**: the only weekData change between release-47 and release-48-2 is Iceland (IS) moving from Monday to Sunday as the first day.
- **date-fns**: tokenizer, protected-token rule and locale week values from date-fns 4.4.0. `P` and `p` expand with the enUS long formats.
- **moment**: tokenizer from moment 2.30.1, en locale. moment 2.31.0 (on npm since 2026-09-15) adds an `eHHmm` token whose output equals `e` followed by `HHmm`. The moment week rule is minDays = 7 + dow - doy, with `e` 0-based.
- **strftime**: [POSIX.1-2024](https://pubs.opengroup.org/onlinepubs/9799919799/functions/strftime.html) conversions in the POSIX locale; `0` and `+` with a width only for `%C %F %G %Y`. glibc `-`, `_`, `^`, `%k`, `%l` and `%P` ([glibc manual](https://sourceware.org/glibc/manual/latest/html_node/Formatting-Calendar-Time.html)). MSVC `#` ([Microsoft Learn](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/strftime-wcsftime-strftime-l-wcsftime-l)). MSVC's locale texts for `%c`, `%x`, `%X` and `%r` are not modeled and show a placeholder.
- **.NET**: [custom format strings](https://learn.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings), InvariantCulture, DateTime of Kind Unspecified. A one-character string is a [standard format](https://learn.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings), so the translator writes `%d`, `%M` and so on when the result is a single specifier. .NET has no week-year and no day-of-year specifier.
- **Not evaluated**: time zone fields, timestamps, and UTS 35 `W F g U b B` are recognized and named but print a placeholder, because they depend on a zone or on data the page does not carry. All arithmetic uses the wall-clock time you typed, with UTC date math.
- **Java and Swift** are not computed. A notes panel cites the [Java SE 25 DateTimeFormatter](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/time/format/DateTimeFormatter.html) and [SimpleDateFormat](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/text/SimpleDateFormat.html) letters and [Apple's archived Data Formatting Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/DataFormatting/Articles/dfDateFormatting10_4.html).

## Third-party notices

| Project | Copyright holder (from its LICENSE) | Licence | What is used |
|---|---|---|---|
| [Unicode CLDR](https://github.com/unicode-org/cldr/blob/release-48-2/LICENSE), release-48-2 | Copyright © 2004-2026 Unicode, Inc. | Unicode License v3 | weekData `firstDay` and `minDays` territory lists (generated from supplementalData.xml), and the English month, weekday, quarter, era and day-period names |
| [moment](https://github.com/moment/moment/blob/2.30.1/LICENSE) 2.30.1 | Copyright (c) JS Foundation and other contributors | MIT | the format tokenizer regular expression, the en long date formats and the week values (dow, doy) of the en, en-gb, en-au, de, fr, pt, is and ar locales |
| [date-fns](https://github.com/date-fns/date-fns/blob/v4.4.0/pkgs/core/LICENSE.md) 4.4.0 | Copyright (c) 2021 Sasha Koss and Lesha Koss https://kossnocorp.mit-license.org | MIT | the format tokenizer regular expressions, the protected-token rule, the enUS long formats and the locale week values (weekStartsOn, firstWeekContainsDate) |

The UTS 35, strftime and .NET behavior was reimplemented from the specifications and measured empirically; no code from those implementations is reproduced here.

This is an independent project. It is not affiliated with or endorsed by Unicode, Microsoft, the moment or date-fns projects, Oracle or Apple. Their names are used only to identify the formats compared.

## License

MIT. Copyright (c) 2026 RBL Industries LLC. See [LICENSE](LICENSE).

## Related

- [unix-timestamp-converter](https://github.com/0xelitesystem/unix-timestamp-converter), a Unix timestamp to UTC, local and ISO 8601 time and back
- [timezone-converter](https://github.com/0xelitesystem/timezone-converter), one moment across many time zones
- [duration-calculator](https://github.com/0xelitesystem/duration-calculator), the exact time between two dates

Part of a catalog of single-file browser tools and plain-language references: [0xelitesystem.github.io](https://0xelitesystem.github.io/). Built by [elitesystem.ai](https://elitesystem.ai).
