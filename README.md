# Condorcet Election Format
**Curent Version:** V1 _(final - 2022-04-27)_
----------------------------
Condorcet Election Format represents an Election, with the candidates and votes data. And relevant parameters to correctly interpret that data. It is specially designed for ranks-based elections like Condorcet methods, STV, or alternative Voting, but can also be used for others like Instant-Runoff.

## Motivation

Representing election parameters and his data is hard. Its **must be** readable and editable by a human, and parsable by a machine without ambiguity.

Various unspecified formats exist, and at least two have more specifications:
* [David Hill format](https://rangevoting.org/TidemanData.html)
_Accept some parameters but don't allow tie on a rank. It is difficult to edit for a human (even for a machine !). It has no known software implementation._
* [Debian format](https://www.debian.org/vote/2021/vote_001_tally.txt) 
_Don't have any parameters. And difficult to read or edit for a human. Specifications are unclear and have changed in time. Only implemented on a Debian-specific program._

This document proposes a new format easy to read and edit by hand. And including many (and extensible) parameters, some of them can be standardized here, some can be proprietary. The votes themselves allow more precision (and particularly tie on a rank) than previous tentatives.

----------------------------

## Summary
1. [Motivation](#motivation)  
1. [Recommended file format](#recommended-file-format)  
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


## Recommended file format

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
* ```#``` Mark a comment until the end of the line. Comment can be everywhere except on parameter lines.
* Empty lines can be everywhere
* ```>``` ```=``` ```;``` ```,``` ```#``` ```/``` are reserved characters and they **must not** be used in any values. But can be used in comments.

### Standard parameters

* ```#/``` At the beginning of the line, are parameters (one per line), followed by the parameter name, then ```:``` without space before, then the parameter value _(should have space(s) before it)_
* Each other non-empty line are vote. Vote lines can are cumulative, one line is a vote and can be combined to line with a quantifier for the same ranking.
* Parameter **must be** specified before the first vote line.

#### ```#/Candidates:```
* **Description:** List of available candidates for this election. Then, vote lines can include other candidates but they **must be** be ignored.
* **Format:** Candidate name separated by semicolon. Candidate name **must be** alphanumerics _(any UTF8 alphabets)_.
* **Optional:** Yes. If parameters are not present, candidates can be parsed directly from votes. But you **should** use this parameter, because it's excluding error (parser implementation, badly formatted vote), and some software can require it.
* * **Notes:** Candidate names **must** be case-sensitive. And spaces are trimmed at the beginning and the end by the parser.
* **Example:** ```#/Candidates: Candidate A ; Candidate B ; Candidate C```


#### ```#/Number of Seats:```
* **Description:** Parameters to apply to some vote computation methods, especially STV or others proportionals methods.
* **Format:** integer
* **Optional:** Yes. 
* **Default Value:** 100. 
* **Example:** ```#/Number of Seats: 42```

#### ```#/Implicit Ranking:```
* **Description:** If lacking candidates on a vote. They are implicitly added to a new last rank.
* **Format:** boolean "true" or "false"
* **Optional:** Yes. 
* **Default Value:** ```true```.
* **Example:** ```#/Implicit Ranking: true```

#### ```#/Voting Methods:```
* **Alias:** ```#/Voting Method:``` (without "s")
* **Description:** Name of the voting method(s) that should be used, multiples methods can be submitted. The names of the methods are currently non standardized. The actual behavior and interpretation of this value may vary from program to program, or be completely ignored depending on context. This parameter is therefore intended as an indication, and requires reference to the documentation of the ingesting program and his implementations choices. 
* **Format:** Method(s) name(s) separated by semicolon, space between methods **can be** used, ingestor **must** trim the value.
* **Optional:** Yes. 
* **Default Value:** ```nothing```.
* **Example:** ```Schulze Margin; Ranked Pairs; Minimax```

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
* Multiples tags **must** be separated by commas. Spaces between commas are not necessary. Spaces are trimmed at the beginning and the end by the parser. Tags **must** be case-sensitive. 
* Tags **must be** alphanumerics _(any UTF8 alphabets)_, they can include space.
* Tags **should be** optional


#### Ranking
* Each vote line **must** have a ranking
* Ranking **must be** case sensitive

* ```>``` is a rank separator. Ranks **should** have space between them (for readability).
* ```=``` is an equality symbol on a rank. **Should** have space before and after him.

* Rank **must not** be empty
* Rank **may** contain candidate(s) who do not take part in this election, then the parser must ignore them and build the right ranking.
* Same candidate **must not** take part on more than one rank per vote.

* Ranking can have the special value ```/EMPTY_RANKING/```, this means a blank _(None of The Above)_ ballot. This should not prevent the interpretation of the vote, if the implicit ranking parameter is activated, as a vote of equality between all candidates. This special keyword replace all the ranking, and cannot be used at a rank position. _Look at the invalid examples section_

#### Special Instructions
* ```*``` is a vote quantifier, it should have one space before and after, it **must be** followed by an integer. This is an optional method to aggregate identical votes on one line.
* ```^``` is a vote weight, it should have one space before none after, it **must be** followed by an integer. This is an optional parameter different from than quantifier because saying that this only vote has more importance than default weight _(1)_

* Weight and quantifier can be chained as follow _(42 different votes have the same ranking and a weight set to 7)_:  
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
Candidate C # Interpreted as Candidate C > Candidate A = Candidate B, because implicit ranking is true (which is also default, but it's better to say it)
Candidate B > Candidate C # Interpreted as Candidate B > Candidate C
```

* _Note that ```#/Number of seats:``` parameter is optional_
* _Note the comment at the end of the lines_

#### Example without implicit ranking as weight
```yaml
# My beautiful election
#/Candidates: Candidate A ; Candidate B ; Candidate C
#/Implicit Ranking: false
#/Weight allowed: false

# Here are the votes data:
Candidate A > Candidate B > Candidate C ^7 *2 # Vote weight is disabled, so ^7 is ignored. Two votes with a weight of 1.
Candidate C>Candidate B # Vote is untouched. When compute pairwise, Candidate C win again Candidate B, no one beats the candidate or achieves a draw.
/EMPTY_RANKING/ * 2 # Two blanks (NOTA) votes.
Candidate B # Vote is valid but does not have any effect on most election methods, especially Condorcet methods.
```

### Invalid

```yaml
# My beautiful election
#/Candidates: Candidate A;Candidate B;Candidate C # On comment
#/Implicit Ranking   : false
```
> Comments are not allowed on parameters lines

```yaml
# My beautiful election
#/Candidates: Candidate A;Candidate B;Candidate C
    #/Implicit Ranking   : false
```
> Parameters must start at the beginning of the line

```yaml
# My beautiful election
#/Candidates: Candidate A;Candidate B;Candidate C

Candidate A < Candidate B > Candidate C
```
> Vote is invalid: ``<``` is not allowed

```yaml
# My beautiful election
#/Candidates: Candidate A;Candidate B;Candidate C

Candidate A > Candidate B > Candidate A
```
> Vote is invalid: ```Candidate A``` is present at first and third rank.

```yaml
# My beautiful election
#/Candidates: Candidate A;Candidate B;Candidate C

Candidate A > Candidate B > Candidate A  * 8 ^7
```
> Vote is invalid: The right order is ```^7 * 8```

```yaml
# My beautiful election
#/Candidates: Candidate A;Candidate B;Candidate C; Candidate D
#/Implicit Ranking: false

Candidate A > Candidate B > /EMPTY_RANKING/
```
> ```/EMPTY_RANKING/``` keyword cannot be used as a rank. The correct ranking is ```Candidate A > Candidate B``` wich will be interpreted as it if implicit ranking parameter is false, and as ```Candidate A > Candidate B > Candidate C = Candidate D``` if implicit parameter is true.

## Questions

__Why not include standard parameters specifying how the results are calculated (methods, variants)?__  
> The purpose of this format is to represent the data to be inserted and the vital parameters for their correct interpretation. It is then up to the user of the program that ingests them to use it according to his needs and abilities.  
If needed, the programs ingesting these data can use additional non-standard properties, allowing them to go further.

__Does the current format allow for methods without ranks, such as majority judgment?__
> Methods like Majority judgment are not supported. But ballot format can evolve in the future without modifying parameters, tags, or quantifier/weight format.

## Software Implementations

|                      Software =>                  |  [Condorcet PHP](https://github.com/julien-boudry/Condorcet) >= v3.3  |
|:-------------------------------------------------:|:-----------------------:|
| Read Condorcet format file or string                |            ✓            |
| Generate Condorcet format file or string            |            ✓            |
| ```#/``` Parameters: Candidates                     |            ✓            |
| ```#/``` Parameter: Number of Seats                 |            ✓            |
| ```#/``` Parameter: Implicit Ranking                |            ✓            |
| ```#/``` Parameter: Voting Methods                  |            ✗            |
| ```#/``` Parameter: Weight Allowed                  |            ✓            |
| ```#``` Comments                                    |            ✓            |
| ```*``` Quantifier                                  |            ✓            |
| ```^``` Weight                                      |            ✓            |
| ```\|\|``` Tags on vote                             |            ✓            |
| Chained ```^``` weight with ```*``` quantifier      |            ✓            |
| Generate ```/EMPTY_RANKING/``` ranking              |            ✓            |
| Importing ```/EMPTY_RANKING/``` ranking             |            ✓            |
| Parse Candidates directly from votes lines          |            ✓ <br>_(since V4.0)_    |
| System: Parsing huge files without memory problems  |            ✓            |
| System: Generate huge files without memory problems |            ✓            |
| Behavior: Parse invalid vote (bad format)           | Line should be skipped.<br>Failed on some cases. |

