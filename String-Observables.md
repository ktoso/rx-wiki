The `StringObservable` class contains methods that represent operators particular to Observables that deal in string-based sequences and streams. These include:

* [**`byLine( )`**](String-Observables#wiki-byline) — converts an Observable of Strings into an Observable of Lines by treating the source sequence as a stream and splitting it on line-endings
* [**`decode( )`**](String-Observables#wiki-decode) — convert a stream of multibyte characters into an Observable that emits byte arrays that respect character boundaries
* [**`encode( )`**](String-Observables#wiki-encode) — transform an Observable that emits strings into an Observable that emits byte arrays that respect character boundaries of multibyte characters in the original strings
* [**`from( )`**](String-Observables#wiki-from) — convert a stream of characters or a Reader into an Observable that emits byte arrays or Strings
* [**`join( )`**](String-Observables#wiki-join) — converts an Observable that emits a sequence of strings into an Observable that emits a single string that concatenates them all, separating them by a specified string
* [**`split( )`**](String-Observables#wiki-split) — converts an Observable of Strings into an Observable of Strings that treats the source sequence as a stream and splits it on a specified regex boundary
* [**`stringConcat( )`**](String-Observables#wiki-stringconcat) — converts an Observable that emits a sequence of strings into an Observable that emits a single string that concatenates them all

*** 

## byLine( )
#### converts an Observable of Strings into an Observable of Lines by treating the source sequence as a stream and splitting it on line-endings
[[images/rx-operators/St.byLine.png]]

*** 

## decode( )
#### convert a stream of multibyte characters into an Observable that emits byte arrays that respect character boundaries
[[images/rx-operators/St.decode.png]]

*** 

## encode( )
#### transform an Observable that emits strings into an Observable that emits byte arrays that respect character boundaries of multibyte characters in the original strings
[[images/rx-operators/St.encode.png]]

*** 

## from( )
#### convert a stream of characters or a Reader into an Observable that emits byte arrays or Strings
[[images/rx-operators/St.from.png]]

*** 

## join( )
#### converts an Observable that emits a sequence of strings into an Observable that emits a single string that concatenates them all, separating them by a specified string
[[images/rx-operators/St.join.png]]

*** 

## split( )
#### converts an Observable of Strings into an Observable of Strings that treats the source sequence as a stream and splits it on a specified regex boundary
[[images/rx-operators/St.split.png]]

*** 

## stringConcat( )
#### converts an Observable that emits a sequence of strings into an Observable that emits a single string that concatenates them all
[[images/rx-operators/St.stringConcat.png]]