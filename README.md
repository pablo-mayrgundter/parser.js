# parser.js
Text parser in Javascript. Formal grammars with recursive rules and first-class JS RegExp-based rules.

# Example
Parse a simple format for asterisms (constellations), found in [Celestiary|https://pablo-mayrgundter.github.io/celestiary/#sun]:

```
const asterisms = `"Ursa Major"
[
[ "Eta UMa" "Zeta UMa" "Epsilon UMa" "Delta UMa" "Gamma UMa" "Beta UMa" "Alpha UMa" "Delta UMa" ]
]

"Ursa Minor"
[
[ "Alpha UMi" "Delta UMi" "Epsilon UMi" "Zeta UMi" "Beta UMi" "Gamma UMi" "Eta UMi" "Zeta UMi" ]
]
`;

let records = [];
let recordName;
let paths = [];
let path = [];
let names = [];
let nameList = [];
const Grammar = {
  'Start': { // List of records
    rule: [ 'Record', [ 'Start', Parser.Terminal ] ]
  },
  'Record': {
    rule: [ 'Name', 'OuterArray' ],
    callback: (state, match) => {
      recordName = names.pop();
      records.push({
          name: recordName,
            paths: paths
            });
      recordName = null;
      paths = [];
      names = [];
      nameList = [];
    }
  },
  'Name': {
    rule: [ /\p{Z}*"([\p{L}0-9 ]+)"\p{Z}*/u ],
    callback: (state, match) => {
      names.push(match[1]);
    }
  },
  'NameList': {
    rule: [ 'Name', [ 'NameList', Parser.Terminal ] ],
    callback: (state, match) => {
      nameList.unshift(names.pop());
      console.log('nameList after: ', nameList);
    }
  },
  'OuterArray': {
    rule: [ /\s*\[\s*/ , 'ListInnerArray' , /\s*\]\s*/ ],
  },
  'ListInnerArray': {
    rule: [ 'Path', [ 'ListInnerArray', Parser.Terminal ] ],
  },
  'Path': {
    rule: [ /\s*\[\s*/ , 'NameList', /\s*\]\s*/ ],
    callback: (state, match) => {
      paths.push(nameList);
      nameList = [];
      console.log('paths after: ', paths);
    }
  }
};

const parser = new Parser();
const parseLength = parser.parse(asterisms, Grammar, 'Start');
console.log(records); // Yields...
/*
[
  {
    "name": "Ursa Major",
    "paths": [
      [
        "Eta UMa",
        "Zeta UMa",
        "Epsilon UMa",
        "Delta UMa",
        "Gamma UMa",
        "Beta UMa",
        "Alpha UMa",
        "Delta UMa"
      ]
    ]
  },
  {
    "name": "Ursa Minor",
    "paths": [
      [
        "Alpha UMi",
        "Delta UMi",
        "Epsilon UMi",
        "Zeta UMi",
        "Beta UMi",
        "Gamma UMi",
        "Eta UMi",
        "Zeta UMi"
      ]
    ]
  }
]
*/
```
