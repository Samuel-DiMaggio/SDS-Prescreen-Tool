# SDS Prescreen Tool with instructions on how to convert a python file to an EXE using Nuitka
This tool utilizes tokenization from the NLP library to search through a given Safety Data Sheet (SDS) to identify chemicals of interest that pertain to federal and international regulatory requirements. This was a project that I worked on as a Regulatory Analyst for a chemical company that specialized on adhesives for consumer and industrial products. The main purpose was to quickly identify or flagg chemicals that appear in a SDS that also are listed on mulitple federal and international regulatory lists. In the first half of this project, will be the python code that I wrote for this project. This contains a simple GUI input and output for a easy user experience. In the second half of this project, there will be a reference guide on  how to convert a python file to a executable file using Nuitka. From my experience in this project, Nuitka was the easiest and fastest way to convert a python file to an executable file that also can be shared with someone else. I recommend if you visited this link for learning how, I recommend jumping to section 2 (eventualy, I will create another respository for only this and provide a link here). There will be additional information about Nuitka located there as well.

# Part 1  
## Section 1: Importing applicable libraries
```
import PySimpleGUI as sg      
import PyPDF2
import re
import spacy
import pandas as pd
import numpy as np
import en_core_web_sm
import os
import sys
```
## Section 2: Creating the desired GUI using PySimpleGUI:
The below code creates the first page of the GUI. The image below that is what is outputted after running the entire script for part 1 in a single cell in a jupyter notebook. I went with a text input box with titles describing what to do. Of course, there are addtional methods in PySimpleGUI for doing the same task. However, changing the input method will also effect how the variables are read, so if considering changing the method here, additional changes in other sections may need to be applied. For more references on PySimpleGUI I recommend following this link: https://pysimplegui.readthedocs.io/en/latest/ 
```
layout = [[sg.Text('Copy and Paste File Name Here add .pdf')], [sg.InputText()],
          [sg.Text('Type list.csv')], [sg.InputText()],
          [sg.Text('Copy and Paste Folder Path')], [sg.InputText()],
          [sg.Submit(), sg.Cancel()]]

window = sg.Window('SDS Prescreening', layout)    

event, values = window.read()    
window.close()
```
![image](https://user-images.githubusercontent.com/47721595/147795535-92d9909f-a016-473e-a4c4-64a50bdb4517.png)

## Section 3: Assigning variables
Note: values with brackets contianing a number indicate which input field from GUI is assigned. Example: values[0] indicates the first text field in the simple GUI. 
```
nlp = en_core_web_sm.load()
filename = values[0]
Important_list = values[1]
Path = values[2]
```
## Section 4: Setting up PDF reader and reading a master list  pertaining to Federal/International Chemical Regulations 
```
pdfFile = open(os.path.join(Path, filename),'rb')
df2 = pd.read_csv(os.path.join(Path, Important_list))
pdfFileReader = PyPDF2.PdfFileReader(pdfFile)
pageCount = pdfFileReader.numPages
output = []
```
## Section 5: Tokenizing the word count
In the below code, for the variable "tokens2", you will notice that its finding all characters until the 3500th character location with the syntax of (\d{1,6}[-]??\d{2}[-]??\d{1}). This is because in the world of chemical regulations every chemical and/or substance is assigned a numerical code. This numerical code is called a CAS number. For instance Formaldehyde's CAS number is 50-00-0. In the section three of a Safety Data Sheet, is where a list of substances that make up finished product that are deemed hazardous and are classified under the global harmonized system (GHS). This location is normally where someone would identify a CAS number. Section 1 of an SDS identifies the product and manufacturer's contact information. Section 2 of the SDS lists hazards classified under GHS. Setting the search to end at the 3500th character location reduces the chances of having additional information that may or may not be needed depending on task. If needed to illustrate this, I have provided SDS examples from two different manufacturers in the files section and screenshots of the first pages of one below the code.
```
for i in range(pageCount):
    pdfPage = pdfFileReader.getPage(i)
    output.append(pdfPage.extractText())
alltexts = ' '.join(output)  
alltexts = re.sub('\n', '', alltexts)
alltexts = re.sub(r'','',alltexts)
doc = nlp(alltexts)
tokens = re.findall("[\w']+", alltexts[:2000])
tokens2 = re.findall("(\d{1,6}[-]??\d{2}[-]??\d{1})", alltexts[:3500])
```
![image](https://user-images.githubusercontent.com/47721595/147798427-b9ba31ea-3454-419c-80a9-235630eff17e.png)
![image](https://user-images.githubusercontent.com/47721595/147798469-0a0c7455-0781-479a-9a16-fd8a34875447.png)

## Section 6: Listing specific words of interest or "flags" that might indicate hazards
As previously mention, in section 2 of a SDS is where a company lists GHS classified hazards for a product. There is many different type of hazards along with categorical indentifiers for each hazard. Prior to 2013, hazards were listed very ambiguously and left a reader uncertain. As a result GHS was created to reduce this ambigous language in section this section, but also create a standard for all Safety Data Sheets. This of course is whether or not a country adapts this as a regulation, so some SDSs may differ for regions. 

If you want to learn more about GHS, I would recommend this link: https://www.ccohs.ca/oshanswers/chemicals/ghs.html 
Also, if you are a curious person and want to know hazards of specific chemicals I would recommend searching through Europes Chemical Agency's database by this link: https://echa.europa.eu/

Note: Once you click and get routed to desired substance page, click on C&L Inventory button located half way down the page. This will list hazards associated with the substance if its not diluted.
```
hazards = ['Flammable', 'flammable', 'Gases', 'gases', 'pyrophoric', 'Pyrophoric', 'Chemically', 
           'unstable', 'Aerosols', 'chemically', 'Unstable', 'aerosols', 'oxidizing', 'aquatic',
           'Oxidizing', 'under', 'pressure', 'liquids', 'Liquids', 'Solids', 'solids', 'sensitisation', 
           'exposure','self','Self', '-', 'heating', 'Organic', 'peroxides','Corrosive', 'metals', 'corrosive', 
           'Desenitized', 'explosives', 'Acute', 'Toxicity', 'toxicity', 'oral','dermal', 'inhale',
           'Oral', 'Dermal', 'Inhalation', 'Skin', 'Corrosion', 'Irritation', 'Serious', 'Eye', 'Damage', 
           'irritation', 'Respiratory', 'Sensitization', 'Germ', 'Cell', 'Mutagenicity', 'Carcinogenicity', 
           'Reproductive', 'Specific', 'target', 'organ', 'STOT','Single', 'Repeated', 'Aspiration', 
           'repeated', 'Hazard','Combustible', 'Dusts', 'Hazardous', 'Ozone', 'Layer', 'Aquatic', 'Chronic', 
           'single', 'Category', '1', '2','2A', '2B', '1A', '1B', '1C', '3', '4', 'Danger', 'Warning',
           'No', 'need','product', 'is', 'classified', 'does', 'not', 'hazardous', 'GHS', 'criteria', 
           'warning']
```
## Section 7: Assigning variables for each column in the master of list (i.e. the list.csv)
This section could have be added earlier, but it didn't really matter too much except for being before the next section. Here I am assigning variables for each column in the list.csv file. Since I created a EXE file at the end, one issue I kept running into to is having the the code just automatically import the file if the location may change after sending the exe somewhere else. The easiest way was to just have the user input the file into the gui and assigned these variables. The list could be updated whenever the user desired or needed too based off of regulations and staying current. Each column belongs to a specific list, such as TSCA is The Toxic Substances Control Act of 1976 outlined by the EPA. Which has roughly around 64 thousand CAS numbers listed, hence why prescreening tool can be helpful. 
```
TSCA = df2['TSCA'].to_list()
HAPS = df2['HAPS'].to_list()
EHS = df2['EHS'].to_list()
COI = df2['COI'].to_list()
RTK = df2['RTK'].to_list()
SVHC = df2['SVHC'].to_list()
RED = df2['RED'].to_list()
EPA = df2['EPA'].to_list()
LTAP = df2['LTAP'].to_list()
SNUR = df2['SNUR List'].to_list()
NES_HAPS = df2['NES HAPS'].to_list()
PBT = df2['PBT\n(WAC 173-333-320)'].to_list()
IARC = df2['IARC'].to_list()
```
## Section 8: Then checking whether a token is contained in any list and/or "flag" hazard word
This section is rather explanatory, mostly just running a for loop and checking whether or not the tokens are listed on any of the lists within the list.csv file or hazard list mentioned in section 6 once the variable is called. When the variable is called the identified substance or flagged hazard token is then assigned into a list for the outputted screen.  
```
check =  any(item in hazards for item in tokens)
check1 =  any(item in TSCA for item in tokens2)
check2 =  any(item in HAPS for item in tokens2)
check3 =  any(item in EHS for item in tokens2)
check4 =  any(item in COI for item in tokens2)
check5 =  any(item in RTK for item in tokens2)
check6 =  any(item in SVHC for item in tokens2)
check7 =  any(item in RED for item in tokens2)
check8 =  any(item in EPA for item in tokens2)
check9 =  any(item in LTAP for item in tokens2)
check10 =  any(item in SNUR for item in tokens2)
check11 =  any(item in NES_HAPS for item in tokens2)
check12 =  any(item in PBT for item in tokens2)
check13 =  any(item in IARC for item in tokens2)
x = (f' There are {pageCount} pages in the file: {filename}')
if check is True:
    x1 = [x for x in hazards if x in tokens]
else:
    x1 = "n/a"
if check1 is True:
    x2 = [x for x in TSCA if x in tokens2]
else:
    x2 = "n/a"
if check2 is True:
    x3 = [x for x in HAPS if x in tokens2]
else:
    x3 = "n/a"
if check3 is True:
    x4 = [x for x in EHS if x in tokens2]
else:
    x4 = "n/a"
if check4 is True:
    x5 = [x for x in COI if x in tokens2]
else:
    x5 = "n/a"
if check5 is True:
    x6 = [x for x in RTK if x in tokens2]
else:
    x6 = "n/a"
if check6 is True:
    x7 = [x for x in SVHC if x in tokens2]
else:
    x7 = "n/a"
if check7 is True:
    x8 = [x for x in RED if x in tokens2]
else:
    x8 = "n/a"
if check8 is True:
    x9 = [x for x in EPA if x in tokens2]
else:
    x9 = "n/a"
if check9 is True:
    x10 = [x for x in LTAP if x in tokens2]
else:
    x10 = "n/a"
if check10 is True:
    x11 = [x for x in SNUR if x in tokens2]
else:
    x11 = "n/a"
if check11 is True:
    x12 = [x for x in NES_HAPS if x in tokens2]
else:
    x12 = "n/a"
if check12 is True:
    x13 = [x for x in PBT if x in tokens2]
else:
    x13 = "n/a"
if check13 is True:
    x14 = [x for x in IARC if x in tokens2]
else:
    x14 = "n/a"
``` 
## Section 9: Outputted pop-up describing what was found
Here the  newly created lists are displayed in a pop-up after clicking submit in the GUI. I have also added the sg.popup_scrolled function so that a user can copy of the information if needed.
```
sg.popup_scrolled("Auto Prescreened information:","-"*100, 
                  "File information: \n", x, "-"*100,
                  "Possible Hazard Stop Words: \n", x1, "-"*100,
                  "Important Lists Evaluation: \n",
                  'TSCA List: ',x2, " ", 
                  "HAPS List: ", x3, " ", 
                  "EHS List: ", x4, " ", 
                  "COI List: ", x5, " ", 
                  "RTK List: ", x6, " ", 
                  "SVHC List: ", x7, " ", 
                  "RED List: ", x8, " ", 
                  "EPA List: ", x9, " ", 
                  "LTAP List: ", x10, " ", 
                  "SNUR List: ", x11, " ",
                  "NES_HAPS List: ", x12, " ",
                  "PBT List: ", x13, " ",
                  "IARC List: ", x14, "-"*100,)
```
## Section 10: What it looks like
Below are two examples of the provided SDS files and how they look on the first GUI page and the outputted popup.

For file: 0101030_Foamstar ST 2412_BASF_08.03.2018_EN.pdf

![image](https://user-images.githubusercontent.com/47721595/147800020-be3fc55f-09a9-4431-9688-ece3722367e5.png)

![image](https://user-images.githubusercontent.com/47721595/147800058-4ed56b5f-5f22-4cf6-8a4e-d46d3dfd76dd.png)

This pop-up basically indicates that there may not be a hazard, however one substance was found to be listed on the TSCA list.

For file: 0110116 MONDUR 1453 - Covestro - 14-JUN-2018 - English.pdf

![image](https://user-images.githubusercontent.com/47721595/147800207-07a10114-2bd5-4182-9350-bc9d3ffc92ed.png)

![image](https://user-images.githubusercontent.com/47721595/147800160-61f250d6-e383-4bde-bbd3-3a389fd5e028.png)

This pop-up indicates that there is most likely a hazard or multiple hazards and multiple substances listed on a few list such as RTK (Right to Know), HAPS (Hazardous air pollutants), and TSCA. 

# Part 2 - How to Convert a Python File to Exe File Using Nuitka

## Section 1: Intro: Why I used Nuitka

After trying to explore and learn a method to convert a python file into a sharable executable file, I stumbled across Nuitka. There isn't really many methods that are as simple to learn and debugg then Nuitka (that I found that is). Most that did seem to work weren't really easy for certain projects. Given my background, as in I am not a computer science major, I had no clue how to convert a python file to an executable file. However, I kept at it trying to learn and debugg my way through different methods. I tried using pyinstaller, auto-py-to-exe, py2exe, but for my project there was always something missing or I had no clue what was missing. Some of the tutorials on the web really didn't go in depth. Some of them were also created early on and lost support for Python3, which in most cases, the only way I find out was through forums. Eventually, I stumbled on a list of Python compliers that mentioned Nuitka. So, I gave it a shot and after some time with some trial and error, I had a more progress than other methods. As I delved into their manual and a few forums, I was able to produce a shareable executible file for this tool.

## Section 2: So What is Nuitka

As described on Nuitka's website: 

"Nuitka is a Python compiler written in Python. Which is also fully compatible with Python2 (2.6, 2.7) and Python3 (3.3 - 3.10). Basically, you feed Nuitka your Python app, it does a lot of clever things, and spits out an executable or extension module."

Link: https://nuitka.net 

Manual: https://nuitka.net/doc/user-manual.html

## Section 3: Steps and how I arrived to getting this to work for me

Note: These steps may vary for each person or project depending on:
1. which libraries are being imported 
2. whether or not the exe file is being shared
3. file size constraints
4. version of certain libraries
5. whether or not certian files are in the right directory (this is a result of a library missing a file that may be located else where)

## Requirements:
Mentioned in the User manual there is some requirements for using Nuitka:

C Compiler: You need a compiler with support for C11 or alternatively for C++03 1
Currently this means, you need to use one of these compilers:

1. The MinGW64 C11 compiler on Windows, must be based on gcc 11.2 or higher. It will be automatically downloaded if no usable C compiler is found, which is the recommended way of installing it, as Nuitka will also upgrade it for you.

2. Visual Studio 2022 or higher on Windows 2, older versions will work but only supported for commercial users. Configure to use the English language pack for best results (Nuitka filters away garbage outputs, but only for English language). It will be used by default if installed.

3. On all other platforms, the gcc compiler of at least version 5.1, and below that the g++ compiler of at least version 4.4 as an alternative.

4. The clang compiler on macOS X and most FreeBSD architectures.

5. On Windows the clang-cl compiler on Windows can be used if provided by the Visual Studio installer.

6. Python: Version 2.6, 2.7 or 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9

## Installing Nuitka:
python -m pip install nuitka

Verify using command python -m nuitka --version

## If applicable, convert .ipynb to .py
If you are using jupyter notebook, you will need to download your .ipynb file as a .py file. To do this open the .ipynb file in juypter notebook. Under the file tab, scroll down to download as, to the right will be a list. Select python (.py). Basically, this will just copy your .ipynb as a .py file. 

## Basics:

1. Use command line 
2. Set or change directory where .py file is located
Example: cd C:\Users\Your_Name\Desktop\folder
3. 

## Some of my Tips
1. Use --help to get 
2. Read the manual when you get lost, however
3. Make sure to enable pluggins, if unsure read through the --help in command line
4. use --plugin-enable=anti-bloat to reduce size of the exe file or unwanted files in the file.dist
5. Be prepared to compile lots of extension modules by using --module some_module.py for every module that is needed or missing. The unknown modules can be determine by trial and error with running the new exe file in Virtual Studio. 
6. Be prepared to add files to the file.dist folder from specific folders. These files aren't located in the correct file location when compiling, this results in an error. The list below were the missing files based off of my imported libraries in my python code:

located in directory:
...\AppData\Roaming\Python\Python38\site-packages\tensorflow\lite
copy over the "experimental folder" to: ...\Desktop\Folder\file.dist\tensorflow\lite

in this directory:
...\anaconda3\Lib\site-packages\thinc\backends
copy over "_custom_kernals.cu" and "_murmur3.cu" to"...\Desktop\folder\file.dist\thinc\backends

in this directory:
...\anaconda3\Lib\site-packages\spacy
copy over "default_config.cfg" and "cli" folders to:  ...\Desktop\folder\file.dist\spacy

in this directory:
...\anaconda3\Lib
copy over the "en_core_web_sm" file to: ...\Desktop\Folder\file.dist

## My end result that got me a sharable Exe file:

python -m nuitka --mingw64 --standalone --follow-imports --plugin-enable=anti-bloat --plugin-enable=numpy --plugin-enable=pylint-warnings --plugin-enable=multiprocessing --plugin-enable=tensorflow --plugin-enable=tk-inter --plugin-enable=torch --plugin-enable=pbr-compat --plugin-enable=pkg-resources --plugin-enable=data-files --plugin-enable=dill-compat --plugin-no-detection --prefer-source-code --assume-yes-for-downloads --show-scons --include-module=srsly.msgpack.util --include-module=cymem --include-module=preshed.maps --include-module=thinc.backends.linalg --include-module=blis --include-module=spacy.parts_of_speech --include-module=spacy.tokens._dict_proxies --include-module=spacy.lexeme --include-module=spacy.lang.norm_exceptions --include-module=spacy.lang.lex_attrs --include-module=spacy.pipeline.transition_parser --include-module=spacy.pipeline._parser_internals.stateclass --include-module=spacy.pipeline._parser_internals.transition_system --include-package-data=PACKAGE_DATA --include-package=spacy --include-package=tensorflow --include-package=thinc --include-package=en_core_web_sm --include-package-data=PATTERN --windows-icon-from-ico=favicon.ico NLP_SDS_Prescreening_os.py
