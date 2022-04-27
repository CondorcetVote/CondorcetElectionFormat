# Condorcet Election Format
**Curent Version:** V1 _(final - 2022-04-27)_
----------------------------

Representing an election parameters and his data is hard. It's must be readable and editable by a humain, and parsable by a machine without ambiguity.

Various unspecified format exist, and at least two have more specification:
* [David Hill format](https://rangevoting.org/TidemanData.html)
_Accept some parameters, but don't allow tie on a rank. He is dificult to edit for a human (even for a machine !). It has no known software implementation._
* [Debian format](https://www.debian.org/vote/2021/vote_001_tally.txt) 
_Don't have any parameter. And dificult to read or editing for a human. Specification is unclear and have changend in time. Only implemented on a Debian specific program._

This document proposes a new format easy to read and edit by hand. And including many (and extensible) parameters, some of them can be standardized here, some can be proprietary. The votes themselves allows more precision, and tie on a rank.

----------------------------

## Recommanded file format

* Charset **must** be ```UTF8```
* Endline **should be** ```Unix "\n"```
* File Extension **should be** ```.cvotes``` in lower case
* Case sensibility is **required** for Candidate name and votes (including tags) but is **optional** for parameters.

## General Form

```yaml
#/Candidates: CandidateA ; CandidateB ; CandidateC
#/Number of Seats: 42
#/Implicit Ranking: true
#/Weight allowed: true

Candidate A > Candidate B > Candidate C * 42
Candidate C > Candidate A = Candidate B ^7
Candidate B = Candidate A > Candidate C
Candidate C
Candidate B > Candidate C
```

* ```#``` Mark a comment until the end of the line
* ```#/``` At the begining of the line, are parameters (one per line), followed by the parameter name, then ```:``` without space before, then the parameter value _(should have space(s) before it)_
* Each others non-empty line are votes. Votes line can are cumulative, one line is on vote and can be combinated to line with quantifier for the same ranking.

* ```*``` is a vote quantifier, it should have one space before and after, it must be followed by an integer. This is an optionnal method to aggragate intical votes on one line.
* ```^``` is a vote weight, it should have one space before none after, it must be followed by an integer. This is an optionnal parameter different than quantifier, because saying that this only vote has more importance than default weight _(1)_

* Weight and quantifier can be chained as follow _(42 differents votes has same ranking and a weight set to 7)_:  
```Candidate A > Candidate B > Candidate C ^7 * 42```

## Standard parameters

## Implementation

|                      Software                     |  Condorcet PHP >= v3.3  |
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
| System: Parsing huge files without memory problem |            V            |
| System: Generate huge file without memory problem |            V            |
| Parse Candidates directly from votes lines        |            X            |

