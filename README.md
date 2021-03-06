[![Build Status](https://travis-ci.org/KarpelesLab/strftime.png?branch=master)](https://travis-ci.org/KarpelesLab/strftime)
[![GoDoc](https://godoc.org/github.com/KarpelesLab/strftime?status.svg)](https://godoc.org/github.com/KarpelesLab/strftime)
[![Coverage Status](https://coveralls.io/repos/github/KarpelesLab/strftime/badge.svg?branch=master)](https://coveralls.io/github/KarpelesLab/strftime?branch=master)

# strftime

Fast strftime in Go with [BCP 47 language tags](https://golang.org/x/text/language).

# Usage

It is either possible to instanciate an object for a given locale (via `strftime.New`) or directly call `strftime.Format` or `strftime.EnFormat`.

## Examples

To simply get a timestamp in English:

```go
fmt.Printf("%s: something happened", strftime.EnFormat(`%c`, time.Now()));
```

Or get a quick result in French:

```go
fmt.Printf("%s: Quelque chose est arrivé", strftime.Format(language.French, `%c`, time.Now()));
```

Or display a result in the appropriate language for a web user:

```go
tags, _, _ := language.ParseAcceptLanguage(r.Header.Get("Accept-Language"))
f := strftime.New(tags...)
f.FormatF(w, `%c`, time.Now());
```

# Description

This version of strftime for Go has multiple goals in mind:

* Support for [BCP 47 language tags](https://golang.org/x/text/language)
* Easy to use
* Ability to just return string or write to a io.Writer
* Be as complete as possible in terms of conversion specifications

## Pattern support

| pattern | description |
|:--------|:------------|
| %A      | national representation of the full weekday name |
| %a      | national representation of the abbreviated weekday |
| %B      | national representation of the full month name |
| %b      | national representation of the abbreviated month name |
| %C      | (year / 100) as decimal number; single digits are preceded by a zero |
| %c      | national representation of time and date |
| %D      | equivalent to %m/%d/%y |
| %d      | day of the month as a decimal number (01-31) |
| %e      | the day of the month as a decimal number (1-31); single digits are preceded by a blank |
| %F      | equivalent to %Y-%m-%d |
| %f      | Microseconds (6 digits) |
| %G      | Year matching the week going by ISO-8601:1988 standards |
| %g      | Two digits representation of %G |
| %H      | the hour (24-hour clock) as a decimal number (00-23) |
| %h      | same as %b |
| %I      | the hour (12-hour clock) as a decimal number (01-12) |
| %j      | the day of the year as a decimal number (001-366) |
| %k      | the hour (24-hour clock) as a decimal number (0-23); single digits are preceded by a blank |
| %l      | the hour (12-hour clock) as a decimal number (1-12); single digits are preceded by a blank |
| %M      | the minute as a decimal number (00-59) |
| %m      | the month as a decimal number (01-12) |
| %n      | a newline |
| %p      | national representation of either "ante meridiem" (a.m.)  or "post meridiem" (p.m.)  as appropriate. |
| %P      | lower-case version of %p |
| %R      | equivalent to %H:%M |
| %r      | equivalent to %I:%M:%S %p |
| %S      | the second as a decimal number (00-60) |
| %s      | Unix Epoch Time timestamp (seconds since January 1st 1970) |
| %T      | equivalent to %H:%M:%S |
| %t      | a tab |
| %U      | the week number of the year (Sunday as the first day of the week) as a decimal number (00-53) |
| %u      | the weekday (Monday as the first day of the week) as a decimal number (1-7) |
| %V      | the week number of the year (Monday as the first day of the week) as a decimal number (01-53) |
| %v      | equivalent to %e-%b-%Y |
| %W      | the week number of the year (Monday as the first day of the week) as a decimal number (00-53) |
| %w      | the weekday (Sunday as the first day of the week) as a decimal number (0-6) |
| %X      | national representation of the time |
| %x      | national representation of the date |
| %Y      | the year with century as a decimal number |
| %y      | the year without century as a decimal number (00-99) |
| %Z      | the time zone name |
| %z      | the time zone offset from UTC |
| %%      | a '%' |

Era modifiers are available. For locales in which there is no era, normal values (without era modifier) are returned.

| pattern | description |
|:--------|:------------|
| %Ec     | national representation of time and date |
| %EC     | name of era the date is in |
| %EX     | national representation of the time |
| %Ex     | national representation of the date |
| %EY     | full era name and year represented in locale |
| %Ey     | year as decimal number in era (if any) or same as %y |

## Why not Go's Format()?

This is a very good question. Go time package's [`Format()`](https://golang.org/pkg/time/#Time.Format) method has a nice, human friendly method to set the format for a date. Yet, this is unfortunately not appropriate when multiple languages are involved, as each language has its own rules in terms of terms ordering and presentation, and may even use different years.

While maybe less human friendly, `strftime()` has a long history and most developers will know how it works and what to expect from it.

## Performances / other libraries

```
// On Linux Gentoo 4.14.13-gentoo
// go version go1.11 linux/amd64
$ go test -tags bench -benchmem -bench .
<snip>
BenchmarkCactus-12        	 1000000	      1697 ns/op	     216 B/op	       7 allocs/op
BenchmarkLeekchan-12      	  500000	      2775 ns/op	    1376 B/op	      22 allocs/op
BenchmarkTebeka-12        	  500000	      3777 ns/op	     288 B/op	      21 allocs/op
BenchmarkJehiah-12        	 1000000	      1555 ns/op	     256 B/op	      17 allocs/op
BenchmarkFastly-12        	  500000	      3774 ns/op	     192 B/op	      11 allocs/op
BenchmarkLestrrat-12      	 1000000	      1333 ns/op	     240 B/op	       3 allocs/op
BenchmarkKarpelesLab-12    	 2000000	       688 ns/op	     240 B/op	       3 allocs/op
PASS
ok  	github.com/KarpelesLab/strftime	11.997s
```

This library is much faster than other libraries for common cases. In case of format pattern re-use, [Lestrrat's implementation](https://github.com/lestrrat-go/strftime) is still faster (but has no locale awareness).

| Import Path                         | Score      | Note                            |
|:------------------------------------|-----------:|:--------------------------------|
| github.com/KarpelesLab/strftime      | 688 ns/op  |                                 |
| github.com/lestrrat-go/strftime     | 1333 ns/op | Using `Format()` (NOT cached)   |
| github.com/jehiah/go-strftime       | 1555 ns/op |                                 |
| github.com/cactus/gostrftime        | 1697 ns/op |                                 |
| github.com/leekchan/timeutil        | 2775 ns/op |                                 |
| github.com/fastly/go-utils/strftime | 3774 ns/op | cgo version on Linux            |
| github.com/tebeka/strftime          | 3777 ns/op |                                 |

Please note that this benchmark only uses the subset of conversion specifications that are supported by *ALL* of the libraries compared.

