# Fund-Count-Bank-Reconciliation
a python program that uses pandas to parse and compare data from two different sources.
import easygui as gui
import pandas as pd
filename = gui.fileopenbox(msg='Please choose the Excel workbook containing the Fund Count and Wells Fargo worksheets.') #select workbook containing FC and WF data
xls = pd.ExcelFile(filename) #import entire spreadsheet
fieldNames = ["Fund Count Worksheet Name", "Wells Fargo Worksheet Name","Ticker Symbols You Want to Exclude From Analysis separated by commas"] #create field names for multientry user input
fieldValues = list(gui.multenterbox(msg='Fill in values for the fields.', title='Enter', fields=(fieldNames))) #create multientry user input box
FC, WF = [fieldValues[i] for i in (0, 1)] #Set 1st and 2nd user input fields equal to vars FC and WF respectively
list = fieldValues[2].split(",") #convert the 3rd user input value(s) to a list
dfFC = pd.read_excel(xls,FC) #put FC worksheet in df named dfFC
dfWF = pd.read_excel(xls,WF) #put WF worksheet in df named dfWF
dfWF = dfWF.replace('\s+', ' ', regex=True) #replaces any instance of multiple spaces with single space in WF df
dfCombined = dfWF[['Symbol']].drop_duplicates() #create new df = Symbol column of WF df and remove duplicates from that df
dfFC.fillna(method='ffill', inplace = True) #forward fill every NaN value in every column in the dataframe with the preceding real value
dfFC.dropna(axis=1, how='all', thresh=None, subset=None, inplace=True) #drop any column with all values = NaN
dfFC.dropna(axis=0, how='all', thresh=None, subset=None, inplace=True) #drop any row with all values = NaN
header = dfFC.iloc[4] #Creates a header variable at row index location 4
dfFC = dfFC[5:] #Resets dataframe equal to row 5 and beyond
dfFC.rename(columns = header, inplace = True) #sets names of columns in the dataframe equal to header
dfFC = dfFC[dfFC['Cost'] != 'Cost'] #Resets FC dataframe removing any rows with values = Cost
dfCombined['FCInstances']=dfCombined['Symbol'].map(dfFC['Ticker symbol'].value_counts()) #counts the number of instances each Ticker Symbol is in the FC df and puts them in the combined df
dfCombined['WFInstances']=dfCombined['Symbol'].map(dfWF['Symbol'].value_counts()) #counts the number of instances each Ticker Symbol is in the FC df and puts them in the combined df
dfCombined = dfCombined.query("FCInstances != WFInstances") #removes any rows where FCInstances and WFInstances are the same
dfCombined.dropna(axis=0, how='all', thresh=None, subset=None, inplace=True) #drop any row with all values = NaN
dfCombined = dfCombined[~dfCombined['Symbol'].isin(list)] #removes rows in df where symbol equals any value in list
dfCombined.to_csv(gui.filesavebox(msg='Please choose where you want to save the csv of results.')) #convert dataframe to csv and have user decide where to save it and what to name it
