# Scraping

Examine the data file first. XML Files are sorted by this
```XML
<element type="attribute">text</element>
```
The red is the element, the yellow/green is the attribute, the grey is the text.

The ilan name data are generally sorted by the following
```XML
<?xml version="1.0" encoding="UTF-8"?>
<TEI>
    <teiheader>
        <!Unimportant stuff>
    </teiheader>
    <text>
        <body>
            <listPerson>
            <!First Person>
            <person>
            </person>
            <!Second Person>
            <person>
            </person>
            <!N persons>
            </listPerson>
        </body>
    </text>
</TEI>
```
Each person file looks like the following
```XML
<person xmlns:local="http://local-functions.uri"
                    xmlns:xi="http://www.w3.org/2001/XInclude"
                    xml:id="Vol-3-E_M-d8e936-person"
                    n="Vol-3-E_M-Egyptian-Males-Arseion-1">
               <sex cert="high">M</sex>
               <persName type="lemma">Arseion</persName>
               <persName type="lemma" xml:lang="en">Arseion</persName>
               <persName>̓Αρσείονος</persName>
               <state type="desc">
                  <note>
                     <ref corresp="#Vol-3-E_M-d8e9565-person" type="relation" n="father-of">Teuphilos (9)’s father</ref>
                  </note>
               </state>
               <state type="findspot">
                  <note>Epitaph, Gizeh?<ref type="note" corresp="#Vol-3-E_M-ftn61"/>de Ricci</note>
               </state>
               <bibl>CIJ1477</bibl>
               <state type="addl-Descr">
                  <note>Jew?<ref type="note" corresp="#Vol-3-E_M-ftn62"/>
                  </note>
               </state>
               <state type="provenance">
                  <note>Egypt</note>
               </state>
               <floruit>–<ref type="note" corresp="#Vol-3-E_M-ftn63"/>
               </floruit>
</person>
```
We essentially loop through each person in listPerson and take the following things

We want the following data points
```XML
               <persName type="lemma">Arseion</persName>
               <persName type="lemma" xml:lang="en">Arseion</persName>
               
               <sex cert="high">M</sex>
               
               <state type="addl-Descr">
                  <note>Jew?<ref type="note" corresp="#Vol-3-E_M-ftn62"/>
                  </note>
               </state>

               <floruit>–<ref type="note" corresp="#Vol-3-E_M-ftn63"/>
               </floruit>

               <state type="provenance">
                  <note>Egypt</note>
               </state>

               <ref type="note" corresp="#Vol-3-E_M-ftn62"/>

               <state type="findspot">
                  <note>Epitaph, Gizeh?<ref type="note" corresp="#Vol-3-E_M-ftn61"/>de Ricci</note>
               </state>
```

To run it, [download VSCode](https://code.visualstudio.com/docs/setup/windows). 
![[Pasted image 20231230123139.png]]
Once you launch VSCode it'll ask for which folder to open. 
Open the folder, and then choose the folder with all the names in there
![[Pasted image 20231230151242.png]]
Create a new file then name it Automation.py
![[Pasted image 20231230151317.png]]
The file structure to use the code looks like the following
```
Folder
- Automation.py
- name-_Abdalago.xml
- name-_Abdolmithabo.xml
- ...
- #the rest of the names
```
![[Pasted image 20231230123233.png]]
It should look like this.
Once you run the code the csv file will show up.

I've re-written the entire code so it looks like this, I've added comments to specify what each portion does
```Python
import os
import xml.etree.ElementTree as ET
import pandas as pd

#Having a Dot means to got through the folder it is in
directory_path = '.'

# The namespace is necessary because it's in the XML file.
namespace = {
    'tei': 'http://www.tei-c.org/ns/1.0',
}

#Initialize the columns of the dataframe
columns = [
    "Name-English", "Name-Ancient", "Sex", "Date", "Findspot", "Note", "Corres", "Volume", "Bible"
]
df = pd.DataFrame(columns=columns)

#Here is where it gets tricky
#We first tell the code to go through every file in the directory specified
#In this particular case we want it to go through every file in the same folder as the code
for folder, _, files in os.walk(directory_path):
    for file in files:
    #Only take the files that end with .xml
        if file.endswith(".xml"):
            #Specify the filepath for the chosen xml file
            file_path = os.path.join(folder, file)

            #Specify the filepath for the code to start scraping
            tree = ET.parse(file_path)
            root = tree.getroot()

            #Specify that we want to loop over every <person></person> tag inside of the xml file 
            for person in root.findall('.//tei:person', namespaces=namespace):

                #Initilalize the placeholders if there isn't an element
                english_name = ancient_name = sex = date = find_spot = note = corresp = volume = n_attribute = "~"


                #Scrape for the names
                #We're telling it to find each <persName> with the tag = "lemma"
                names = person.findall('tei:persName[@type="lemma"]', namespaces=namespace)
                if names:
                    # If the lang = 'en' then we store it into the english_names
                    #               <persName type="lemma" xml:lang="en">Herod</persName>
                    english_names = [name.text for name in names if name.get('{http://www.w3.org/XML/1998/namespace}lang') == "en"]
                    # If the lang =/= 'en' then we store it into the ancient_names
                    #               <persName type="lemma">̔Ηρώδης</persName>
                    ancient_names = [name.text for name in names if name.get('{http://www.w3.org/XML/1998/namespace}lang') != "en"]
                    english_name = english_names[0] if english_names else "~"
                    ancient_name = ancient_names[0] if ancient_names else "~"

                #Scrape for the <sex> tag, then we take the text inside of it.
                #Ie.
                #               <sex cert="high">M</sex>
                sex_element = person.find("tei:sex", namespaces=namespace)
                sex = sex_element.text if sex_element is not None else "~"

                #We're basically doing the same thing for bibl
                #               <bibl>BJ1:562;AJ17:21 (Schalit,NB, 52)</bibl>
                bible_elements = person.findall('tei:bibl', namespaces=namespace)
                bible = ', '.join([b.text for b in bible_elements if b.text]) if bible_elements else "~"

                #Same thing for floruit
                #Ie.  <floruit notBefore="0001" notAfter="0099">1stC CE<ref type="note" corresp="#Vol-1-G_M-ftn586"/></floruit>
                floruit_element = person.find("tei:floruit", namespaces=namespace)
                date = floruit_element.text if floruit_element is not None else "~"

                #The findspot is the same as well
                #We're scraping for the <state> tage with the type = 'findspot' then to get the note inside of it
                 #               <state type="findspot">
                 #                     <note>Coins, weight-stone- Tiberias, inscription- Athens</note>
                 #               </state>
                state_findspot_element = person.find('tei:state[@type="findspot"]', namespaces=namespace)
                if state_findspot_element is not None:
                    find_spot_element = state_findspot_element.find("tei:note", namespaces=namespace)
                    find_spot = find_spot_element.text if find_spot_element is not None else "~"

                #We're finding the additional description
                #We're scraping for the <state> tag with the type = "addl-Descr" inside of it
                #Once we find that we simply need to find the text inside of the note
                #               <state type="addl-Descr">
                #                   <note>Second name<ref type="note" corresp="#Vol-1-G_M-ftn589"/>
                #                   </note>
                #               </state>
                state_addl_descr_element = person.find('tei:state[@type="addl-Descr"]', namespaces=namespace)
                if state_addl_descr_element is not None:
                    note_element = state_addl_descr_element.find('tei:note', namespaces=namespace)
                    note = note_element.text if note_element is not None else "~"

                #This is pretty much useless now. 
                #It scarpes for the Volume by getting the xml:id
                #            <person xmlns:local="http://local-functions.uri"
                #                    xmlns:xi="http://www.w3.org/2001/XInclude"
                #                    xml:id="Vol-1-G_M-d2e8399-person"
                #                    n="Vol-1-G_M-Herod-5">
                xml_id = person.get('{http://www.w3.org/XML/1998/namespace}id')
                volume = xml_id if xml_id is not None else "~"

                #This scrapes for the content inside of the corresp tag inside of the ref 
                #This code gets all of the corresp tags and then picks one and from there it simply adds it to the corresp tag
                #Note: This code assumes that between all of the corresp tags that the major pieces of information is consistent
                #For example Take a look at the below code
#            <person xmlns:local="http://local-functions.uri"
#                    xmlns:xi="http://www.w3.org/2001/XInclude"
#                    xml:id="Vol-1-G_M-d2e8441-person"
#                    n="Vol-1-G_M-Herod-6">
#               <persName type="lemma">̔Ηρώδης</persName>
#               <persName xml:lang="gr">Ηρωδου<ref type="note" corresp="#Vol-1-G_M-ftn587"/>
#               <sex cert="high">M</sex>
#               <persName xml:lang="checkme">—</persName>
#               <state type="desc">
#                  <note>
#                     <ref corresp="#Vol-1-S-G_M-d2e2925-person"
#                          type="relation"
#                          n="grandson-of">Phasael (1)’s grandson</ref>
#                     <ref type="note" corresp="#Vol-1-G_M-ftn591"/>
#                  </note>
#               </state>
#               <state type="findspot">
#                  <note>—</note>
#               </state>
#               <bibl>AJ18:131 (Schalit,NB, 52)</bibl>
#               <state type="addl-Descr">
#                  <note>Second name<ref type="note" corresp="#Vol-1-G_M-ftn589"/>
#               </state>
#               <floruit>4 BCE- 38 CE<ref type="note" corresp="#Vol-1-G_M-ftn590"/>
#               </floruit>
#            </person>
#The four corresp tags have different ending values, but the same beginning values
# - corresp="#Vol-1-G_M-ftn587"
# - corresp="#Vol-1-G_M-ftn591"
# - corresp="#Vol-1-G_M-ftn589"
# - corresp="#Vol-1-G_M-ftn590"
#The code simply picks the first one it finds
                corresp_element = person.find('.//tei:ref[@type="note"]', namespaces=namespace)
                corresp = corresp_element.get('corresp') if corresp_element is not None and corresp_element.get('corresp') else "~"

                #We append all of the collected data into a row of the dataframe
                df.loc[len(df.index)] = [
                    english_name,
                    ancient_name,
                    sex,
                    date,
                    find_spot,
                    note,
                    corresp,
                    volume,
                    bible,
                ]

                #Prints the data so we know it's running
                print(f"{english_name},{ancient_name},{sex},{date},{find_spot},{note},{corresp},{volume},{bible}")

# Once everything is done, the dataframe is written to CSV
df.to_csv('4th-trial.csv', index=False)
```


# Cleaning Data

So this is the main scraping algorithm.
It'll give us a .csv file which we can use for further data gathering

We go into a Jupyter notebook to continue the data processing. A Jupyter notebook is similar to an R Notebook but with Python instead.

```Python
import pandas as pd
import numpy as np

data = pd.read_csv("4th-trial.csv")
data
```
![[Pasted image 20231230133922.png]]
We first load the data as a pandas dataframe.

The following code erases from the back of the Corres component until it hits a -
So for example
- Before: Vol-2-S-G_M-ftn70
- After: Vol-2-S-G_M
Once we did that we're asking it to display each unique value in the "Corres" column.
```Python
data['Corres'] = data['Corres'].apply(lambda x: x.rsplit('-', 1)[0] if '-' in x else x)

data['Corres'].unique()
```
```
array(['#Vol-4-I_M', '#Vol-4-I_F', '#Vol-4-A_M', '#Vol-4-S-H_M', '#Vol-2-B_M', '#Vol-2-Vol-1-ADD-B_M', '#Vol-4-B_M', '#Vol-3-B_M', '#Vol-4-S-H_F', '#Vol-4-A_F', '#Vol-3-P_M', '#Vol-2-G_M', '#Vol-3-G_M', '#Vol-1-G_M', '#Vol-1-S-H_M', '#Vol-2-S-H_M', '#Vol-2-Vol-1-ADD-P_M', '#Vol-3-S-H_M', '#Vol-2-Vol-4-ADD-S-G_M', '#Vol-2-Vol-4-ADD-S-H_M', '#Vol-2-S-G_M', '#Vol-1-B_M', '#Vol-3-S-G_M', '#Vol-3-S-G_F', '#Vol-4-B_F', '#Vol-1-S-G_M', '#Vol-1-B_F', '#Vol-3-B_F', '#Vol-2-Vol-1-ADD-B_F', '#Vol-2-Vol-4-ADD-A_F', '#Vol-4-Vol-3-Add-B_M', '#Vol-2-Vol-4-ADD-A_M', '#Vol-3-L_M', '#Vol-3-E_M', '#Vol-3-G_F', '#Vol-2-Vol-1-ADD-G_M', '#Vol-1-G_F', '#Vol-2-G_F', '#Vol-2-L_M', '#Vol-3-L_F', '#Vol-4-L_M', '~', '#Vol-1-L_F', '#Vol-4-L_F', '#Vol-1-L_M', '#Vol-2-Vol-1-ADD-L_M', '#Vol-2-Vol-1-ADD-G_F', '#Vol-4-G_M', '#Vol-3-E_F', '#Vol-1-P_M', '#Vol-4-S-G_F', '#Vol-2-Vol-1-ADD-S-H_F', '#Vol-2-Vol-4-ADD-B_M', '#Vol-1-S-H_F', '#Vol-2-L_F', '#Vol-4-Vol-3-Add-G_M', '#Vol-4-Vol-3-Add-L_M', '#Vol-2-Vol-1-ADD-S-G_M', '#Vol-4-G_F', '#Vol-3-Add-G_M', '#Vol-4-S-G_M', '#Vol-2-P_M', '#Vol-2-Vol-1-ADD-S-G_F', '#Vol-2-S-H_F', '#Vol-1-S-G_F', '#Vol-3-P_F', '#Vol-2-B_F', '#Vol-2-Vol-1-ADD-L_F', '#Vol-4-Vol-3-Add-G_F', '#Vol-3-S-H_F', '#Vol-1-Ha_Names', '#Vol-2-Vol-3-ADD-B_M', '#Vol-3-Add-B_M', '#Vol-2-Vol-4-ADD-B_F', '#Vol-3-Add-L_M', '#Vol-3-Add-L_F', '#Vol-1-P_F', '#Vol-4-Vol-3-Add-E_F', '#Vol-4-Vol-3-Add-L_F', '#Vol-2-P_F', '#Vol-2-S-G_F'], dtype=object)
```

We want to have the Vol-2-Vol-3-ADD, Vol-1-Vol-2-ADD, .....etc.
```Python
import pandas as pd

#I borrowed some code from the documentation which used df as the dataframe instead of data
df = data

#For each of the following ones we're doing a regex check to see whether or not the 'Corres' column has the following Regex Expression
#Example:
# r'Vol-(2|1)-Vol-(1|2)-(ADD|Add)'
#The regex expression looks for
# Vol-(2 or 1)-Vol-(1 or 2)-(ADD or Add)

#Vol-2-Vol1-ADD
#Vol-3-Vol1-ADD
#Vol-4-Vol1-ADD
#Vol-3-Vol2-ADD
#Vol-4-Vol2-ADD
#Vol-4-Vol3-ADD
vol2vol1_add_pattern = r'Vol-(2|1)-Vol-(1|2)-(ADD|Add)'
vol3vol1_add_pattern = r'Vol-(3|1)-Vol-(1|3)-(ADD|Add)'
vol4vol1_add_pattern = r'Vol-(4|1)-Vol-(1|4)-(ADD|Add)'
vol3vol2_add_pattern = r'Vol-(3|2)-Vol-(2|3)-(ADD|Add)'
vol4vol2_add_pattern = r'Vol-(4|2)-Vol-(2|4)-(ADD|Add)'
vol4vol3_add_pattern = r'Vol-(3|4)-Vol-(3|4)-(ADD|Add)'

#We apply this to each of the combinations 
df['Vol-2-Vol-1-ADD'] = df['Corres'].str.contains(vol2vol1_add_pattern, regex=True)
df["Vol-3-Vol-1-ADD"] = df['Corres'].str.contains(vol3vol1_add_pattern, regex=True)
df["Vol-4-Vol-1-ADD"] = df['Corres'].str.contains(vol4vol1_add_pattern, regex=True)
df["Vol-3-Vol-2-ADD"] = df['Corres'].str.contains(vol3vol2_add_pattern, regex=True)
df["Vol-4-Vol-2-ADD"] = df['Corres'].str.contains(vol4vol2_add_pattern, regex=True)
df["Vol-4-Vol-3-ADD"] = df['Corres'].str.contains(vol4vol3_add_pattern, regex=True)

df
```
![[Pasted image 20231230135552.png]]
![[Pasted image 20231230135538.png]]

```Python
#This is the section of the code which finds if something is in a particular volume
data = df

#This code checks if the 'Corres' row has the words contianing Vol-1,Vol-2,Vol-3,Vol-4
#If it is true then it'll append True to the new columns
data["Volume 1"] = data["Corres"].str.contains("Vol-1")
data["Volume 2"] = data["Corres"].str.contains("Vol-2")
data["Volume 3"] = data["Corres"].str.contains("Vol-3")
data["Volume 4"] = data["Corres"].str.contains("Vol-4")

data
```
![[Pasted image 20231230140917.png]]


We now want to extract the following from the corres
```Python
#B = Biblical
#S-H = Semitic Hebrew
#S-G = Semitic Greek
#G = Greek
#L = Latin
#P = Persian

Biblical = r'B'
SemeticHebrew = r'S-H'
SemeticGreek = r'S-G'
Greek =  r'(?<!S-)G'
Latin = r'L'
Arabian = r'A(?![dD])'
Iranian = r'I'
Egyptian = r'E'
Persian = r'P'


data['Biblical'] = df['Corres'].str.contains(Biblical, regex=True)
data['Semetic-Hebrew'] = df['Corres'].str.contains(SemeticHebrew, regex=True)
data['Semetic-Greek'] = df['Corres'].str.contains(SemeticGreek, regex=True)
data["Greek"] = df['Corres'].str.contains(Greek, regex=True)
data["Latin"] = df['Corres'].str.contains(Latin, regex=True)
data["Persian"] = df['Corres'].str.contains(Persian, regex=True)
data["Arabian"] = df["Corres"].str.contains(Arabian, regex = True)
data["Egyptian"] = df["Corres"].str.contains(Egyptian, regex = True)
data["Iranian"] = df["Corres"].str.contains(Iranian, regex = True)
data
```
![[Pasted image 20231230145946.png]]

Exceptions to Note:
- Since B, L, I, E, and P are words that are not contained in each other nor are they contained in Vol-1,Vol-2,Vol-3,Vol-4,ADD they are unique and exclusive and therefore there are no edge cases
- Note: L is case sensitive and different from l in Vol. 
- Edge cases are
    - G since S-G contains a capital G inside of it
    - A since Add and ADD has a capital A inside of it

To counter the edge cases
- G checks only if there contains a G without a S- behind it
- A checks for an A if there is no d afterwards
    - (Note: I realized I forgot to check if there is a capital D afterwards as well in the rough draft. This was subsequently corrected)
    - Before: Arabian = r'A(?!d)'
    - After: Arabian = r'A(?![dD])'




I added some code that proves that l and L are two different cases
```Python
import pandas as pd

# Assuming you have a DataFrame df with a column 'Corres'
data = pd.DataFrame()

# Sample DataFrame
df = pd.DataFrame({'Corres': ['Hello', 'world', 'Lion', 'Tall', 'small']})

L = r'L'
l = r'l'

# Creating columns to check if 'L' and 'l' are present
data['L'] = df['Corres'].str.contains(L, regex=True)
data['l'] = df['Corres'].str.contains(l, regex=True)

print(data)
```
![[Pasted image 20231230143107.png|200]]

We drop Volume since it was unnecessary
```Python
data.drop('Volume', axis = 1, inplace = True)
```

We save it to a csv file
```Python
data.to_csv("Final-4th-Trial.csv")
```


# Checking for Troublesome Data Points

Learning from my last mistake of shipping it once it's done we're doing our due dilligance in checking the data
I've written code that prints the unique outputs of each of the columns, starting from the 7th column
```Python
for column in df.iloc[:, 6:]:
    unique_values = df[column].unique()
    print(f"Unique values in {column}: {unique_values}")
```
We're particularly interested in the following data points
```
Unique values in Vol-2-Vol-1-ADD: [False True] 
Unique values in Vol-3-Vol-1-ADD: [False] 
Unique values in Vol-4-Vol-1-ADD: [False] 
Unique values in Vol-3-Vol-2-ADD: [False True] 
Unique values in Vol-4-Vol-2-ADD: [False True] 
Unique values in Vol-4-Vol-3-ADD: [False True] 
Unique values in Volume 1: [False True] 
Unique values in Volume 2: [False True] 
Unique values in Volume 3: [False True] 
Unique values in Volume 4: [ True False] 
Unique values in Biblical: [False True] 
Unique values in Semetic-Hebrew: [False True] 
Unique values in Semetic-Greek: [False True] 
Unique values in Greek: [False True] 
Unique values in Latin: [False True] 
Unique values in Persian: [False True] 
Unique values in Arabian: [False True] 
Unique values in Egyptian: [False True] 
Unique values in Iranian: [ True False]
```

The most common issue in these particular test cases comes from poorly written code that doesn't catch what is necessary resulting in a single value output.
Ie. 
```Python
#Added a typo in the bottom
vol2vol1_add_pattern = r'Vol-(2|1)-Vol -(1|2)-(ADD|Add)'

df['Vol-2-Vol-1-ADD'] = df['Corres'].str.contains(vol2vol1_add_pattern, regex=True)
```

Due to the space it will return false for everything.

Noticing we might have an issue in the
- Vol-3-Vol-1-ADD
- Vol-4-Vol-1-ADD

We can manually check the data itself by using the data-viewer and manually inputting both cases
- Vol-3-Vol-1 / Vol-1-Vol-3
- Vol-4-Vol-1 / Vol-1-Vol-4
![[Pasted image 20231230150850.png]]

To see if they exist and we missed them.

| Names       | Results                              |
| ----------- | ------------------------------------ |
| Vol-4-Vol-1 | ![[Pasted image 20231230151441.png]] |
| Vol-1-Vol-4 | ![[Pasted image 20231230151510.png]] |
| Vol-3-Vol-1 | ![[Pasted image 20231230151528.png]] |
| Vol-1-Vol-3            | ![[Pasted image 20231230151547.png]]                                     |

As one can see that for every case it doesn't exist therefore the code seems to be working correctly.

Other checks of diligence
- ![[Pasted image 20231230151726.png]]
Adding the True to the Statements and seeing if the Corres is correct.
I've done this for each one and can confirm that they are correct as far as it seems.

