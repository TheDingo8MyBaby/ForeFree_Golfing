# ForeFree ~ Putting the Tee in Vulnerability


-----


## Introduction
In my investigation of NFC (Near Field Communication) Cards at a Driving Range, I explored their operational intricacies, security considerations, and data handling procedures.

This write-up aims to provide a comprehensive understanding for a diverse readership by offering explanations that cover both conceptual and technical aspects.

-----


### Equipment
For this research project, I used the following items/devices:
- Proxmark3 Easy (512kb) running the [Iceman Firmware](https://github.com/RfidResearchGroup/proxmark3).
- Gen1a Magic Mifare Classic Cards
- Driving Range issued NFC Card

The Proxmark3 Easy is a hardware platform used for interacting with and investigating High and Low-frequency RFID credentials such as the ones used at this facility.


-----


### The Card in Question
The NFC Card used at the facility was powered by the [Mifare Classic 1K](https://www.nxp.com/docs/en/data-sheet/MF1S50YYX_V1.pdf), a widely deployed High-Frequency chipset. These chips are prevalent across various industries due to their convenience. 

A simple scan revealed its identity: 

``` hf search ```

![PM3](https://i.imgur.com/QOQyxmK.png)

```
Result:
- UID: 1E E6 5D 01
- ATQA: 00 04
- SAK: 08 [2]
- Card Type: Mifare Classic 1K
```

The Mifare Classic chipset, while widely used, possesses exploitable cryptographic vulnerabilities, which were a primary focus of my investigation.


-----




``` hf mf autopwn ```

```
Result:
- Successful key recover for each sector
```

![Autopwn](https://i.imgur.com/CHmy6lc.png)


-----


### Double Trouble
To begin my investigation I added $20 worth of credit to my card and then dumped the credentials of the card into a ```.eml``` file. I then made a replica of my card using a Gen1a Magic Mifare Classic Card, which is a special type of mifare classic, used in duplication operations as it allows all of its data to be changed to reflect that of another mifare classic including its Unique Identifier. 

After several swings, decreasing the available credit on the Original Driving Range Card, I applied the duplicated card to the reader, which showed I still had $20 left. 
This was interesting, as this means that the card itself must be locally storing the credit on the card and not utilizing any kind of back-end validation system to track balances.

> While this is the intended purpose of Mifare Classic cards, due to the insecurity of having the credits stored locally in plain ASCII, untracked digitally, and the issues that arise from users losing their card, most places that would use these cards, instead use a value system.
>> IE: Arcades tend to use this type of card solely as an identifier which is then used to look up the user's information on a back-end system where the value is stored and retrieved as necessary.


-----


## Original Card:
  ![NFC Card Image 1](https://i.imgur.com/mmmR12y.png)

## Duplicate Card:
  ![Cloned NFC Card Image](https://i.imgur.com/WWLpQLh.png)


-----


### Data Transfer and Replication
With the Proxmark3 Easy at my disposal, I performed successful scans and data extraction from the NFC Driving Range Card.


The extracted data formed the foundation for the subsequent duplications of the card's information onto a blank Gen1a Magic Mifare Classic Card, setting the stage for comprehensive testing.


-----


### Discovering the 'Source of Truth'
An alarming realization surfaced during my early tests at the Driving Range. I had discovered that the 'source of truth', for determining the card's fund balance, resided solely within the card itself.


This revelation contrasted sharply with my initial assumption, that such data would be centrally administered, either on a server or within the Driving Range's equipment.


-----


### Local Values
When analyzing the memory of the original credential with that of the duplicate, it revealed discrepancies in blocks 5 and 6 of the Mifare Classics.


These differences strongly suggested that any stored value resided in these blocks.


To further analyze this vulnerability, I converted the hexadecimal contents of the blocks to ASCII


-----


#### Original Card
- Current credit according to the ball machine: $10.22
  - Block 5: `303031302E3232343831202020202020`
  - Block 6: `32303233303931343138323133312020`

- After Conversion
  - Block 5: `0010.22481`
  - Block 6: `20230914182131`

#### Duplicate Card
- Current Credit According to ball machine: $12.31
  - Block 5: `303031322E3331343831202020202020`
  - Block 6: `32303233303931343138313331312020`

- After Conversion
  - Block 5: `0012.31481`
  - Block 6: `20230914181311`


-----


### Block 5 Examination

Block 5 of the original card was where I had focused the majority of my attention.

The following hexadecimal data was examined in a hex editor:
  - Block 5 [Raw]: `303031302E3232343831202020202020`
  - Block 5 [ASCII]: `0010.22481`


![NFC Card Image 2](https://i.imgur.com/DPuxjwA.png)


The ASCII interpretation of the hexadecimal data exposed the text "0010.22481." This held vital information:

- 10.22: Representing the card balance which indicates a remaining balance of $10.22.
- 48: A numeric value corresponding to the tee height setting.
- The significance of the ending "1" within this data element is presently under review and requires further investigation.


----------


## Editing and Testing NFC Cards
Now I knew that the data was stored locally on the cards and where the information was in the data blocks.


I decided that the next step would be to start editing the card's data for testing purposes. This would allow me to determine just how serious of a vulnerability this could potentially be.


> In the following phase of the attempted exploitation, the objective was to modify the card's data and write it onto a new card for the purpose of testing.

### Data Dump
The card's data was successfully extracted into a ```.eml``` file, allowing for specific line modifications. 
The initial data extraction presented as follows:

- Block 0: 1EE65D01A4080400032C48089DFCFB1D
- Block 1: 00000000000000000000000000000000
- Block 2: 00000000000000000000000000000000
- Block 3: FFFFFFFFFFFFFF078069FFFFFFFFFFFF
- Block 4: 303031302E3232343831202020202020
- Block 5: 303432302E3639343230202020202020
- Block 6: 32303233303931343138323133312020
- Block 7: DDBBCC221166FF078069DDBBCC221166
- (and so on...)


-----


### Editing Block 5
Specifically, block 5 of the card's data was modified to read as follows:

```30 34 2 0 2E 36 39 4 2 0 20 20 20 20 20 20```

Adjusting the above value via the hexadecimal editor resulted in the text "```0420.69420```". This indicated the card should now operate as follows:

- ```420.69```: Representing the card balance, indicating a remaining balance of $420.69
- ```42```: Representing the tee tee height setting.
- The significance of the ending "0" within this data element is presently under review and requires further investigation.


-----


### Writing to the cloned card
``` hf mf cload -f hf-mf-1EE65D01-dump.eml ```

I utilized the following command to write the edited data, including the updated block 5 changes, to the cloned card:

![copy](https://i.imgur.com/KVjRXqk.png)


-----


### Verifying the cloned card
``` hf search ```

Result:
- UID: 1E E6 5D 01
- ATQA: 00 04
- SAK: 08 [2]

The important thing here, is that the UID of the cloned card, matches the UID of the original card.

![search](https://i.imgur.com/LpLsVg3.png)


-----


### Testing the Edited Card
With the newly cloned and edited card in hand, I went back to the Driving Range to test if my theory was correct.

And to my surprise, I was greeted with the following when I slotted in the custom card.
- Currency Availability: The ASCII text '420.69' denotes the available currency balance on the card.
- Tee Height: The numeric value '42' denotes the tee height configuration.


-----


### Supporting Visual Evidence
For visual documentation of the testing process, please refer to the image provided below:

![Testing the Edited NFC Card](https://i.imgur.com/Hz31Aez.png)


-----


### Secondary Testing of the Edited Card
Now that I know that I am able to change the amount that is on the card, along with other minor settings, such as the tee height. I then wanted to test if the system had any type of security in place, to only allow "whitelisted" UIDs.


### DNGO Hex Test
In order to test if UIDs were being Whitelisted, I decided to change the UID of a Duplicated Card that I knew worked originally.  I changed the UID from:
```
[+]  UID: 1E E6 5D 01
[+] ATQA: 00 04
[+]  SAK: 08 [2]
```

to:

```
[+]  UID: 44 4E 47 4F
[+] ATQA: 00 04
[+]  SAK: 08 [2]
```

![DNGO Hex NFC Card](https://i.imgur.com/g0uRYD2.png)

With testing concluded, it was made clear to me, that this site allows for any UID to work on their system, assuming that the other blocks of the card match up to their specifications.


This means that if I had malicious intent, I could, in theory, load a card with $999.99 and use a UID that is unknown to the system, and drive for free.


What's also an issue, is that if the UID is discovered and they decide to blacklist it, I can just create a new UID and continue to do the same thing indefinitely.


-----


## Incorporating Additional Insights
### Mifare Classic Card Vulnerabilities
In the world of NFC card security, the Mifare Classic 1K chipset is a recurring concern. Its inherent vulnerabilities make it susceptible to exploits, allowing attackers to recover sector keys and access data within the card. Tools such as the Proxmark client make this process relatively straightforward. The use of ```hf mf autopwn``` is particularly noteworthy, as it can potentially, automatically recovers keys for each sector.


-----


### Local Storage of Card Data
One of the notable revelations during my investigation was that the local storage of credit data, was on the card itself in ASCII.


Unlike systems that rely on centralized servers for balance validation, the Mifare Classic card stores this information internally. While this approach aligns with its intended use, it introduces security risks, especially if the card is lost or stolen.


-----


## Conclusion
In conclusion, this investigation into NFC Cards at The Driving Range unveiled significant vulnerabilities inherent in the widely used Mifare Classic 1K chipset.


These vulnerabilities, such as key recovery and the ability to locally edit card data, shed light on the risks associated with relying on local storage for card information, raising concerns about data security and exploitation potential.


The findings underscore the importance of adopting more secure practices in card-based systems and reevaluating the reliance on outdated technologies susceptible to manipulation.
