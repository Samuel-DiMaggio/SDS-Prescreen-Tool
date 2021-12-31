# SDS Prescreen Tool with instructions on how to convert a python file to an EXE using Nuitka
This tool utilizes tokenization from the NLP library to search through a given Safety Data Sheet (SDS) to identify chemicals of interest that pertain to federal and international regulatory requirements. This was a project that I worked on as a Regulatory Analyst for a chemical company that specialized on adhesives for consumer and industrial products. The main purpose was to quickly identify or flagg chemicals that appear in a SDS that also are listed on mulitple federal and international regulatory lists. In the first half of this project, will be the python code that I wrote for this project. This contains a simple GUI input and output for a easy user experience. In the second half of this project, there will be a reference guide on  how to convert a python file to a executable file using Nuitka. From my experience in this project, Nuitka was the easiest and fastest way to convert a python file to an executable file that also can be shared with someone else. I recommend if you visited this link for learning how, I recommend jumping to section 2. There will additional information about Nuitka located there.

## Part 1  
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
Note: values with brackets contianing a number indicate which input field from GUI is assigned. Example: values[0] indicates the first field. 
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
In the below code, for the variable "tokens2", you will notice that its finding all characters until the 3500th character location with the syntax of (\d{1,6}[-]??\d{2}[-]??\d{1}). This is because in the world of chemical regulations every chemical and/or substance is assigned a numerical code. This numerical code is called a CAS number. For instance Formaldehyde's CAS number is 50-00-0. In the third section of a Safety Data Sheet, is where a list of substances that make up finished product that are deemed hazardous and are classified under the global harmonized system (GHS). Here is normally where someone would identify a CAS #. Setting the search to end at the 3500th character location reduces the chances of having additional information that may or may not be needed depending on task.
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
Listing specific words of interest or "flags" that might indicate hazards
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
Assigning variables based off of the columns from the csv that indicate which regulations
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
Then checking whether a token is contained in any list and/or "flag" hazard word
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
Outputted pop-up describing what was found
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
