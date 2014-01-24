# Meteor Incremental CSV Package

Process a CSV file incrementally on either the client or server. Useful when a file is large or you can't (or don't want to) store it first.

## Installation:
1. `npm install -g meteorite` (if not already installed)
2. `mrt add incr-csv`

## Documentation:

### Constructors:

```
IncrmentalCSV(function onRecord(row, index));
IncrmentalCSV(object options);
```

OnRecord is called once for each row and is passed the parsed record as an array of strings and the zero-based ordinal index of the row.

Available options and defaults are:
```
  quoteCharacter:  '"',                       // length must be 1
  fieldSeparator:  ',',                       // length must be 1
  recordSeparator: '\r\n',                    // length must be 1 or 2
  onRecord:        function (row, index) {}
```
### Methods:

```
  push(string);
```

Push a string into the parser for processing. The string should be a valid CSV fragment, but does not need to be a complete row.

```
  finish();
```

Since the CSV specification allows the last line of the input to be unterminated calling finish() will flush any remaining data.

## Example:

Incrementally parse a CSV downloaded from some URL:

```javascript
var csv = new IncrementalCSV(
  function onrecord(rec, idx) {
    // process your data here.
    // idx is zero-based, skip if there's a header.
    console.log(idx, "->", rec);
  }
);

url = 'https://extranet.who.int/tme/generateCSV.asp?ds=dictionary';
csv.push(HTTP.get(url).content);
```

Incrementally parse a CSV file uploaded using Chris Mather's [EventedMind File Uploader](https://github.com/EventedMind/meteor-file).

```javascript
if (Meteor.isServer) {
  // array of parsers for multi-file upload.
  var csvs = {};

  Meteor.methods({
    'uploadFile': function (file) {
      // create parser on first block
      if(file.start === 0) {
        csvs[file.name] = new IncrementalCSV(
          function(rec, idx) {
            // process your data here.
            // idx is zero-based, skip if there's a header.
            console.log(idx, "->", rec);
          }
        );
      }

      // convert data to a buffer and
      // send it as a string to the parser
      var buffer = new Buffer(file.data);
      csvs[file.name].push(buffer.toString());

      // end of file. flush the parser and cleanup
      if (file.size === file.end) {
        csvs[file.name].finish();
        delete csvs[file.name];
      }
    }
  });
}
```

## Limitations

* onRecord will return a simple array of values; there's no way to detect the header line or manually specify fields so as to have key-value pairs (objects) returned
* newlines in values - TBD

## Credit:

The parser is adapted from cvsToArray v2.1 by Daniel Tillin:

https://code.google.com/p/csv-to-array/
