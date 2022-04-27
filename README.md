# Condorcet Election Format
**Curent Version:** V1 _(final - 2022-04-27)_
----------------------------

Condorcet Election Format represents an Election, with the candidates and votes data. And relevant parameters to correctly interpret that data.

## Motivation

Representing an election parameters and his data is hard. It's **must be** readable and editable by a humain, and parsable by a machine without ambiguity.

Various unspecified format exist, and at least two have more specification:
* [David Hill format](https://rangevoting.org/TidemanData.html)
_Accept some parameters, but don't allow tie on a rank. He is dificult to edit for a human (even for a machine !). It has no known software implementation._
* [Debian format](https://www.debian.org/vote/2021/vote_001_tally.txt) 
_Don't have any parameter. And dificult to read or editing for a human. Specification is unclear and have changend in time. Only implemented on a Debian specific program._

This document proposes a new format easy to read and edit by hand. And including many (and extensible) parameters, some of them can be standardized here, some can be proprietary. The votes themselves allows more precision (and in particular tie on a rank) than previous tentatives.

----------------------------

## Summary
1. [Motivation](#motivation)  
1. [Recommanded file format](#recommanded-file-format)  
1. [Format Specifications](#format-specifications)  
  A. [Structure](#format-specifications)    
  B. [Standard parameters](#standard-parameters)     
  C. [Votes Lines](#votes-lines)  
    * [Tags](#tags)
    * [Ranking](#ranking)
    * [Special Instructions](#special-instructions)
1. [Examples](#examples)  
   * [Valid](#valid)
   * [Invalid](#invalid)
1. [Questions](#questions)  
1. [Software Implementations](#software-implementations)  

----------------------------


## Recommanded file format

* Charset **must be** ```UTF8```
* Endline **should be** ```Unix "\n"```
* File Extension **should be** ```.cvotes``` in lower case
* Case sensibility is **required** for Candidate name and votes (including tags) but is **optional** for parameters.

## Format Specifications

```yaml
# My beautiful election
#/Candidates: Candidate A ; Candidate B ; Candidate C
#/Number of Seats: 42
#/Implicit Ranking: true
#/Weight allowed: true

# Here the votes datas:
Candidate A > Candidate B > Candidate C * 42
Candidate C > Candidate A = Candidate B ^7
Candidate B = Candidate A > Candidate C
Candidate C
Candidate B > Candidate C
```


* First block **must be** the ```Parameter Block``` then the ```Votes Block```
* ```#``` Mark a comment until the end of the line. Comment can be evrywhere except on parameters lines.
* Empty lines can be everywhere


### Standard parameters

* ```#/``` At the begining of the line, are parameters (one per line), followed by the parameter name, then ```:``` without space before, then the parameter value _(should have space(s) before it)_
* Each others non-empty line are votes. Votes line can are cumulative, one line is on vote and can be combinated to line with quantifier for the same ranking.
* Parameter **must be** specified before the first vote line.

#### ```#/Candidates:```
* **Description:** List of available candidates for this election. Then, vote lines can include others candidates but they **must be** be ignored.
* **Format:** Candidate name separated by semilicon. Candidate name **must be** alphanumerics _(any UTF8 alphabets)_ they can include space. ```>```, ```=```, ```;```, ```,```, ```#``` are strictly prohibited.
* **Optional:** Yes. If parameters is not present, candidates can be parsed directly from votes. But you **should** use this parameter, because it's excluding error (parser implementation, badly formatted vote), and some softwares can required it.
* * **Notes:** Candidate names **must** be case-sensitive. And space are trimmed at the beginning and the end by the parser.
* **Example:** ```#/Candidates: Candidate A ; Candidate B ; Candidate C```

#### ```#/Number of Seat:```
* **Description:** Parameters to apply to some vote computation methods, especially STV or others proportionals methods.
* **Format:** integer
* **Optional:** Yes. 
* **Default Value:** 100. 
* **Example:** ```#/Number of Seat: 42```

#### ```#/Implicit Ranking:```
* **Description:** If lacking candidates on a vote. They are implicitly added ton a new last rank.
* **Format:** boolean "true" or "false"
* **Optional:** Yes. 
* **Default Value:** ```true```.
* **Example:** ```#/Implicit Ranking: true```

#### ```#/Weight Allowed:```
* **Description:** Allowing votes to have a weight _(look at the vote lines section)_. If false, all lines have a weight equal to 1 even if others are specified.
* **Format:** boolean "true" or "false"
* **Optional:** Yes. 
* **Default Value:** ```false```
* **Example:** ```#/Weight Allowed: true```

### Votes Lines

``` (TAGS ||) Ranking (^Weight) (* Quantifier)```

#### Tags
``` tag 1 , tag2 || Candidate A > Candidate B > Candidate C * 42```

* Tags **must** always come first
* Multiples tags **must** be separated by commas. Space between commas are not necessary. Space are trimmed at the beginning and at the end by the parser. Tags **must** be case-sensitive. 
* Tags **must be** alphanumerics _(any UTF8 alphabets)_, they can include space. ```>```, ```=```, ```;```, ```,```, ```#``` are strictly prohibited.
* Tags **should be** optionnal


#### Ranking
* Each vote line **must** have a ranking
* Ranking **must be** case sensitive
* ```>``` is a rank separator. Ranks **should** have space between them (for readability).
* Rank **must not** be empty
* Rank **may** contain candidate who do not take part in this election, then parser must ignore them and build the right ranking.
* Same candidate **must not** take part on more than one rank per vote.

* ```=``` is an equality symbol on a rank. **Should** have space before and after him.

#### Special Instructions
* ```*``` is a vote quantifier, it should have one space before and after, it **must be** followed by an integer. This is an optionnal method to aggragate intical votes on one line.
* ```^``` is a vote weight, it should have one space before none after, it **must be** followed by an integer. This is an optionnal parameter different than quantifier, because saying that this only vote has more importance than default weight _(1)_

* Weight and quantifier can be chained as follow _(42 differents votes has same ranking and a weight set to 7)_:  
```Candidate A > Candidate B > Candidate C ^7 * 42```


## Examples
### Valid

#### Example with tags and implicit ranking
```yaml
# My beautiful election
#/Candidates: Candidate A;Candidate B;Candidate C
#/Implicit Ranking: true
#/Weight allowed: true

# Here the votes datas:
Candidate A > Candidate B > Candidate C * 42
julien@condorcet.vote , signature:55073db57b0a859911 || Candidate A > Candidate B > Candidate C # Same as above, so there will be 43 votes with this ranking. And tags are registered by the software if able.
Candidate C > Candidate A = Candidate B ^7 * 8 # 8 votes with a weight of 7.
Candidate B = Candidate A > Candidate C
Candidate C # Interpreted as Candidate C > Candidate A = Candidate B, because implicit ranking is true (wich is also default, but it's better to say it)
Candidate B > Candidate C # Interpreted as Candidate B > Candidate C
```

* _Note that ````#/Number of seats:``` parameter is optional_
* _Note the comment at the end of the lines_


#### Example without implicit ranking as weight
```yaml
# My beautiful election
#/Candidates: Candidate A ; Candidate B ; Candidate C
#/Implicit Ranking: false
#/Weight allowed: false

# Here the votes datas:
Candidate A > Candidate B > Candidate C ^7 *2 # Vote weight is disable, so ^7 is ignored. Two vote with weight of 1.
Candidate C>Candidate B # Vote is untouched. When compute pairwise, Candidate C win again Candidate B, no one beats the candidate or achieves a draw.
Candidate B # Vote is valid, but not have any effect on most election method, especially Condorcet methods.
```


### Invalid

## Questions

__Why not include standard parameters specifying how the results are calculated (methods, variants)?__  
> The purpose of this format is to represent the data to be inserted and the vital parameters for their correct interpretation. It is then up to the user of the program that ingests them to use it according to his needs and abilities.  
If needed, the programs ingesting these data can use additional non-standard properties, allowing to go further.

## Software Implementations

|                      Software =>                  |  [Condorcet PHP](https://github.com/julien-boudry/Condorcet) >= v3.3  |
|:-------------------------------------------------:|:-----------------------:|
| Read Condorcet format file or string              |            V            |
| Generate Condorcet format file or string          |            V            |
| ```#/``` Parameters: Candidates                   |            V            |
| ```#/``` Parameter: Number of Seat                |            V            |
| ```#/``` Parameter: Implicit Ranking              |            V            |
| ```#/``` Parameter: Weight Allowed                |            V            |
| ```#``` Comments                                  |            V            |
| ```*``` Quantifier                                |            V            |
| ```^``` Weight                                    |            V            |
| ```\|\|``` Tags on vote                           |            V            |
| Chained ```^``` weight with ```*``` quantifier    | X <br>_unknown behavior_|
| Parse Candidates directly from votes lines        |            X            |
| System: Parsing huge files without memory problem |            V            |
| System: Generate huge file without memory problem |            V            |
| Comportement: Parse invalid vote (bad format)     | Line should be skipped.<br>Failed on some case. |

