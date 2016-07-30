##Sports Day Program
#Harry Baines

#Modules
from tkinter import *
from tkinter import ttk
from tkinter import filedialog
from os import listdir
from os.path import isfile, join
import sqlite3 as lite
import csv
import os
import datetime

class database_opts:
    def __init__(self):
        #Open database file in program files
        self.dbName = StringVar()
        self.directory = os.path.dirname(os.path.realpath(__file__))
        self.onlyfiles = [f for f in listdir(self.directory) if isfile(join(self.directory, f))]
        for file in self.onlyfiles:
            if file.endswith('.db'):
                self.databaseName = file
                self.openDatabase(self.databaseName)
                self.dbName.set('Database File: ' + self.databaseName)
                break

    def openDatabase(self,name):
        #Open database
        self.con = lite.connect(name)
        self.cur = self.con.cursor()

    def importPupils(self):
        #Import pupil csv
        myMessage = messagebox.askyesno(title='Confirm',message='''Would you like to replace the current list of pupils in the system with the new file?
                                                                   \nAll the current pupils and their results will be erased.''')
        if myMessage == 1:
            self.clearPupilsTable()
            self.clearResultsTable()
            self.file = filedialog.askopenfile(mode='r',filetypes=[("CSV files","*.csv")])
            if self.file != None:
                self.values = self.file.read().split("\n")
                for value in self.values:
                    if value != "":
                        self.separate=value.split(",")
                        if self.separate[0] == 'Forename':
                            self.separate.clear()
                        else:
                            #add pupil to database
                            self.cur.execute("INSERT INTO Pupil(forename,surname,gender,ageGroup,houseName) VALUES (?,?,?,?,?)",
                                             (self.separate[0],self.separate[1],self.separate[2],self.separate[3],self.separate[4]))
                            self.con.commit()
                messageImported = messagebox.showinfo(title='File Imported',message='Your file has successfully been imported.')


    #Result Procedures 
    def createResultsTable(self):
        #create result table
        self.cur.execute('''CREATE TABLE IF NOT EXISTS Result(resultID INTEGER PRIMARY KEY autoincrement,
                         resultVal INTEGER,
                         pupilID INTEGER,
                         eventID INTEGER)''')
        
    def clearResultsTable(self):
        #clear result table
        self.cur.execute("DROP TABLE IF EXISTS Result")
        self.createResultsTable()

    def insertResult(self,resultVal,pupilIDSelectEdit,eventVal):
        #add a result
        self.cur.execute("INSERT INTO Result(resultVal,pupilID,eventID) VALUES(?,?,?)", (resultVal,pupilIDSelectEdit,eventVal,))
        self.con.commit()

    def removeResult(self,pupilID,eventID):
        #delete a result
        self.cur.execute('''DELETE FROM Result WHERE Result.pupilID=? AND
                            Result.eventID=?''', (pupilID,eventID,))
        self.con.commit()

    def resultsByEvent(self,eventToSearch):
        #find results for an event
        self.cur.execute('''SELECT Pupil.pupilID, Pupil.forename, Pupil.surname, Pupil.houseName, Result.resultVal
                         FROM Pupil INNER JOIN Result ON Pupil.pupilID = Result.pupilID
                         WHERE Result.eventID = ? ORDER BY Result.resultVal''', (eventToSearch,))
        self.results = self.cur.fetchall()

    def checkRepeats(self):
        #check for an existing result
        self.checkForRepeat = self.cur.execute('''SELECT Pupil.pupilID,Pupil.forename,Pupil.surname,Pupil.houseName,
                                               Result.eventID,Result.resultVal FROM Pupil
                                               INNER JOIN Result ON Pupil.pupilID = Result.pupilID
                                               WHERE Result.eventID=?''',(self.eventToSearch,))
        self.pupilCheck = self.checkForRepeat.fetchall()


    #Pupil Procedures
    def createPupilsTable(self):
        #create pupil tableß
        self.cur.execute('''CREATE TABLE IF NOT EXISTS Pupil(pupilID INTEGER PRIMARY KEY autoincrement,
                         forename TEXT,
                         surname TEXT,
                         gender TEXT,
                         ageGroup TEXT,
                         houseName TEXT)''')
        
    def clearPupilsTable(self):
        #clear pupil table
        self.cur.execute("DROP TABLE IF EXISTS Pupil")
        self.createPupilsTable()
        self.clearResultsTable()

    def insertPupil(self,forename,surname,gender,age,house):
        #add a pupil
        self.cur.execute("INSERT INTO Pupil(forename,surname,gender,ageGroup,houseName) VALUES (?,?,?,?,?)", (forename,surname,gender,age,house,))
        self.con.commit()

    def updatePupilDetails(self,newForename,newSurname,newGender,newageGroup,newHouse,existingID):
        #update a pupils details
        self.cur.execute("UPDATE Pupil SET forename=?,surname=?,gender=?,ageGroup=?,houseName=? WHERE pupilID=?",
                         (newForename,newSurname,newGender,newageGroup,newHouse,existingID,))
        self.con.commit()

    def removePupil(self,pupil_idDel):
        #delete a pupil
        self.cur.execute("DELETE FROM Pupil WHERE pupilID=?", (pupil_idDel,))
        self.cur.execute("DELETE FROM Result WHERE pupilID=?", (pupil_idDel,))
        self.con.commit()

    def searchForPupilR(self,surname,ageGroup,gender):
        #search for pupils on result screen
        self.cur.execute('''SELECT pupilID,forename,surname,ageGroup,houseName,gender FROM Pupil WHERE surname=? AND ageGroup=?
                         AND gender=? ORDER BY surname''',(surname,ageGroup,gender,))
        self.pupSearchVals = self.cur.fetchall()

    def searchForPupilP(self,gender,ageGroup,houseName):
        #search for pupils on pupil screen
        self.cur.execute('''SELECT Pupil.pupilID,Pupil.forename,Pupil.surname FROM Pupil WHERE gender=? AND ageGroup=? AND houseName=?''',
                         (gender,ageGroup,houseName,)) 
        self.pupilSearchValuesPupils = self.cur.fetchall()

    def orderFore(self,gender,ageGroup,houseName):
        #order pupils by forename
        self.cur.execute('''SELECT Pupil.pupilID,Pupil.forename,Pupil.surname FROM Pupil WHERE gender=? AND ageGroup=? AND houseName=?
                         ORDER BY forename''',(gender,ageGroup,houseName,))

    def orderSur(self,gender,ageGroup,houseName):
        #order pupils by surname
        self.cur.execute('''SELECT Pupil.pupilID,Pupil.forename,Pupil.surname FROM Pupil WHERE gender=? AND ageGroup=? AND houseName=?
                         ORDER BY surname''',(gender,ageGroup,houseName,)) 
        

    #Event Procedures
    def createEventsTable(self):
        #create event table
        self.cur.execute('''CREATE TABLE IF NOT EXISTS Event(eventID INTEGER PRIMARY KEY autoincrement,
                         eventName TEXT,
                         ageGroup TEXT,
                         gender TEXT,
                         event_type TEXT,
                         result_type TEXT,
                         compName TEXT)''')
        
    def clearEventsTable(self):
        #clear event table
        self.cur.execute("DROP TABLE IF EXISTS Event")
        self.clearResultsTable()
        self.createEventsTable()
        
    def insertEvent(self,event,ageGroup,gender,eventType,resultType,cupComp):
        #add an event
        self.cur.execute('''INSERT INTO Event(eventName,ageGroup,gender,event_type,result_type,compName) VALUES(?,?,?,?,?,?)''',
                         (event,ageGroup,gender,eventType,resultType,cupComp))
        self.con.commit()

    def removeEvent(self,name,age,gender,cupComp):
        #delete an event
        self.cur.execute("DELETE FROM Event WHERE eventName=? AND ageGroup=? AND gender=? AND compName=?", (name,age,gender,cupComp,))
        self.con.commit()
    
    def eventAndResultType(self,eventToSearch):
        #find events by event/result type
        self.eAndRTypeArray = []
        for i in self.eventArraySeparate:
            if eventToSearch == (i[1] + ' ' + i[2] + ' ' + i[3]):
                self.eAndRTypeArray.append(i[4])
                self.eAndRTypeArray.append(i[5])

    def findAllAgeFromEvents(self,age):
        #age groups from events
        self.cur.execute("SELECT * FROM Event WHERE Event.ageGroup=?", (age,))
        self.allEvents = self.cur.fetchall()


    #House Procedures
    def createHouseTable(self):
        #create house table
        self.cur.execute('''CREATE TABLE IF NOT EXISTS House(houseID INTEGER PRIMARY KEY autoincrement,
                         houseName TEXT)''')
        
    def clearHouseTable(self):
        #clear house table
        self.cur.execute("DROP TABLE IF EXISTS House")
        self.createHouseTable()
        
    def insertHouse(self, house):
        #add a house
        self.cur.execute("INSERT INTO House(houseName) VALUES(?)", (house,))
        self.con.commit()

    def deleteHouse(self, house):
        #delete a house
        self.cur.execute("DELETE FROM House WHERE houseName=?", (house,))
        self.con.commit()


    #Age group procedures
    def createAgeGroupTable(self):
        #create age group table
        self.cur.execute("CREATE TABLE IF NOT EXISTS AgeGroup(ageGroup TEXT PRIMARY KEY)")
        
    def clearAgeGroupTable(self):
        #clear age group table
        self.cur.execute("DROP TABLE IF EXISTS AgeGroup")
        self.createAgeGroupTable()

    def insertAgeGroup(self,ageGroupVal):
        #add age group
        self.cur.execute("INSERT INTO AgeGroup(ageGroup) VALUES(?)", (ageGroupVal,))
        self.con.commit()

    def deleteAgeGroup(self, ageGroupVal):
        #delete age group
        self.cur.execute("DELETE FROM AgeGroup WHERE ageGroup=?", (ageGroupVal,))
        self.con.commit()


    #Cup Competition Procedures
    def createCupCompTable(self):
        #create cup competition table
        self.cur.execute("CREATE TABLE IF NOT EXISTS CupComp(compName TEXT PRIMARY KEY)")
        
    def clearCupCompTable(self):
        #clear cup competition table
        self.cur.execute("DROP TABLE IF EXISTS CupComp")
        self.createCupCompTable()

    def insertCupComp(self,cupCompetition):
        #add cup competition
        self.cur.execute("INSERT INTO CupComp(compName) VALUES(?)", (cupCompetition,))
        self.con.commit()

    def deleteCupComp(self,cupCompetition):
        #delete cup competition
        self.cur.execute("DELETE FROM CupComp WHERE compName=?", (cupCompetition,))
        self.con.commit()
        

    #Clear All Data
    def clearAllData(self):
        myMessage = messagebox.askyesno(title='Clear All',message='Are you sure you want to clear everything in the system?')
        if myMessage == 1:
            myMessage2 = messagebox.askyesno(title='Clear All',message='Are you sure? You cannot undo this.')
            if myMessage2 == 1:
                self.clearHouseTable()
                self.clearEventsTable()
                self.clearPupilsTable()
                self.clearAgeGroupTable()
                self.clearResultsTable()
                self.clearCupCompTable()
                confirmClear = messagebox.showinfo(title='Clear All',message='Everything has been cleared. The program will now close.')
                self.master.destroy()


    #Read In Values
    def readInHouses(self):
        self.houseArray = []
        self.cur.execute("SELECT houseName FROM House")
        self.houseValues = self.cur.fetchall()
        for i in self.houseValues:
            self.houseArray.append(i[0])
        #single letter
        self.houses = []
        for house in self.houseValues:
            self.houses.append(house[0][0])

    def readInPupils(self):
        self.allPupilTotal = []
        self.pupilValues = self.cur.execute("SELECT pupilID FROM Pupil")
        for i in self.pupilValues:
            self.allPupilTotal.append(i)
            
    def readInEvents(self):
        self.eventArrayFullName = []
        self.eventArraySeparate = []
        self.resultTypeArray = []
        self.cur.execute("SELECT eventID,eventName,ageGroup,gender,event_type,result_type,compName FROM Event")
        self.eventValues = self.cur.fetchall()
        for i in self.eventValues:
            self.eventArrayFullName.append(i[1] + ' ' + i[2] + ' ' + i[3])
            self.eventArraySeparate.append(i)
            self.resultTypeArray.append(i[4])

    def readInResults(self):
        self.resultArray = []
        self.cur.execute("SELECT resultID FROM Result")
        self.resultValues = self.cur.fetchall()
        for i in self.resultValues:
            self.resultArray.append(i)

    def readInAgeGroups(self):
        self.ageGroupArray = []
        self.cur.execute("SELECT ageGroup FROM AgeGroup")
        self.ageGroupValues = self.cur.fetchall()
        for i in self.ageGroupValues:
            self.ageGroupArray.append(i[0])

    def readInCupComps(self):
        self.cupCompArray = []
        self.cur.execute("SELECT compName FROM CupComp")
        self.cupCompValues = self.cur.fetchall()
        for i in self.cupCompValues:
            self.cupCompArray.append(i[0])

    def readInChampions(self,ageGroup,gender):
        #eligible pupils to be champions initially
        self.cur.execute('''SELECT eventID,eventName,ageGroup,gender,event_type FROM Event WHERE Event.event_type <> 'Relay' AND Event.ageGroup = ?
                            AND Event.gender = ?''', (ageGroup,gender,))
        self.eventsChamp = self.cur.fetchall()
        self.cur.execute("SELECT pupilID FROM Pupil WHERE ageGroup=? AND gender=?", (ageGroup,gender,))
        self.pupilChamp = self.cur.fetchall()


    #Search For Events by X
    def findEventsByAG(self,ageGroup):
        #by age group
        self.readInEvents()
        self.eventsFound = []
        self.eventNames = []
        self.eventsFoundSep = []
        for i in self.eventValues:
            if i[2] == ageGroup:
                self.eventsFound.append(i[1] + ' ' + i[2] + ' ' + i[3] + ' ' + i[6][0])
                self.eventsFoundSep.append(i)

    def findEventsByG(self,gender):
        #by gender
        self.readInEvents()
        self.eventsFound = []
        self.eventsFoundSep = []
        for i in self.eventValues:
            if i[3] == gender:
                self.eventsFound.append(i[1] + ' ' + i[2] + ' ' + i[3] + ' ' + i[6][0])
                self.eventsFoundSep.append(i)

    def findEventsByEType(self,eType):
        #by event type
        self.readInEvents()
        self.eventsFound = []
        self.eventNames = []
        self.eventsFoundSep = []
        for i in self.eventValues:
            if i[4] == eType:
                self.eventsFound.append(i[1] + ' ' + i[2] + ' ' + i[3] + ' ' + i[6][0])
                self.eventNames.append(i[1] + ' ' + i[2] + ' ' + i[3])
                self.eventsFoundSep.append(i)

    def findEventsByCriteria(self,eType,aGroup):
        #by event type
        self.readInEvents()
        self.eventsFound = []
        self.eventNames = []
        self.eventsFoundSep = []
        for i in self.eventValues:
            if i[4] == eType and i[2] == aGroup:
                self.eventsFound.append(i[1] + ' ' + i[2] + ' ' + i[3] + ' ' + i[6][0])
                self.eventNames.append(i[1] + ' ' + i[2] + ' ' + i[3])
                self.eventsFoundSep.append(i)

    def findEventsByCComp(self,cComp):
        #by cup competition
        self.readInEvents()
        self.eventsFound = []
        self.eventsFoundSep = []
        for i in self.eventValues:
            if i[6] == cComp:
                self.eventsFound.append(i[1] + ' ' + i[2] + ' ' + i[3] + ' ' + i[6][0])
                self.eventsFoundSep.append(i)

    def findEventsByAll(self,ageGroup,gender,eventType,cupComp):
        #by all
        self.readInEvents()
        self.eventsFound = []
        self.eventsFoundSep = []
        for i in self.eventValues:
            if i[2] == ageGroup and i[3] == gender and i[4] == eventType and i[6] == cupComp:
                self.eventsFound.append(i[1] + ' ' + i[2] + ' ' + i[3] + ' ' + i[6][0])
                self.eventsFoundSep.append(i)




                
class mainWindow(database_opts):
    #System colour
    rgb_light_green = "#%02x%02x%02x"%(194,239,194)
    def __init__(self,master):
        #Run constrctor for database class
        database_opts.__init__(self)
        
        #Create all tables if don't exist
        self.createHouseTable()
        self.createEventsTable()
        self.createPupilsTable()
        self.createAgeGroupTable()
        self.createCupCompTable()
        self.createResultsTable()
        
        #Define window and set time
        self.master = master
        self.master.title('Main Window')
        self.master.geometry('750x570+190+50')
        self.master.configure(background=self.rgb_light_green)
        self.currentTime = StringVar()
        self.currentTime.set('Date: ' + str(datetime.date.today()))

        
        #Home Window
        #Frames
        self.mainScreenTop = Frame(self.master,pady=20,padx=10,bg=self.rgb_light_green)
        self.mainScreenTop.pack(side=TOP)
        self.mainScreenLeft = Frame(self.master,padx=30,bg=self.rgb_light_green)
        self.mainScreenLeft.pack(side=LEFT)

        #Top frame - title, info and buttons
        self.labelMain = Label(self.mainScreenTop, text='AKS Sports Day Program',font=('Courier',25),
                               bg=self.rgb_light_green).grid(row=0,column=0)
        self.createdBy = Label(self.mainScreenTop, text='created by Harry Baines',font=('Courier',15),
                               bg=self.rgb_light_green).grid(row=1,column=0)
        logo = PhotoImage(file="images/AKS_Logo_1.gif")
        self.labelForLogo = Label(self.mainScreenTop, image=logo,width=160,height=160).grid(row=0,column=5,padx=30,rowspan=3)
        self.logoLabel = Label(image=logo)
        self.logoLabel.image = logo
        self.labelHomePage = Label(self.mainScreenTop, text='Home Page',font=('Courier',22),
                                   bg=self.rgb_light_green,relief=RAISED).grid(row=2,column=0)
        self.labelUpdated = Label(self.mainScreenTop, textvariable=self.currentTime,
                                  bg=self.rgb_light_green).grid(row=3,column=0,pady=10)
        self.labelDb = Label(self.mainScreenTop, textvariable=self.dbName,
                                  bg=self.rgb_light_green).grid(row=3,column=5,pady=15)
        self.refreshHomeButton = Button(self.mainScreenTop, text='Refresh', command=self.refreshListboxes,
                                        highlightbackground=self.rgb_light_green,
                                        width=10).grid(row=4,column=5,pady=5)
        self.helpButton = Button(self.mainScreenTop, text='Help', command=self.helpProg,
                                 highlightbackground=self.rgb_light_green,
                                 width=10).grid(row=4,column=0,pady=5)
        self.quitButton = Button(self.mainScreenTop, text='Quit', command=self.quitProgram,
                                 highlightbackground=self.rgb_light_green,
                                 width=8).grid(row=5,column=0)

        #Left frame - database details
        self.labelHouseNum = Label(self.mainScreenLeft, text='Houses',bg=self.rgb_light_green).grid(row=1,column=0)
        self.houseList = Listbox(self.mainScreenLeft, width=13, height=10)
        self.houseList.grid(row=2,column=0,rowspan=7,padx=40)
        self.labelNumPupils = Label(self.mainScreenLeft, text='Number of Pupils:',
                                    bg=self.rgb_light_green).grid(row=2,column=1)
        self.pupilNumList = Listbox(self.mainScreenLeft, width=8,height=2)
        self.pupilNumList.grid(row=2,column=2)
        self.labelNumEvents = Label(self.mainScreenLeft, text='Number of Events:',
                                    bg=self.rgb_light_green).grid(row=3,column=1)
        self.eventNumList = Listbox(self.mainScreenLeft, width=8,height=2)
        self.eventNumList.grid(row=3,column=2)
        self.labelNumResults = Label(self.mainScreenLeft, text='Number of Results:',
                                     bg=self.rgb_light_green).grid(row=4,column=1)
        self.resultNumList = Listbox(self.mainScreenLeft, width=8,height=2)
        self.resultNumList.grid(row=4,column=2)
        self.labelNumHouses = Label(self.mainScreenLeft, text='Number of Houses:',
                                    bg=self.rgb_light_green).grid(row=5,column=1)
        self.housesNumList = Listbox(self.mainScreenLeft, width=8,height=2)
        self.housesNumList.grid(row=5,column=2)
        self.labelNumCupComps = Label(self.mainScreenLeft, text='Number of Cup Competitions:',
                                      bg=self.rgb_light_green).grid(row=6,column=1)
        self.cupCompsNumList = Listbox(self.mainScreenLeft, width=8,height=2)
        self.cupCompsNumList.grid(row=6,column=2)
        self.labelAgeGroups = Label(self.mainScreenLeft, text='Number of Age Groups:',
                                    bg=self.rgb_light_green).grid(row=7,column=1,padx=20)
        self.ageGroupsNumList = Listbox(self.mainScreenLeft, width=8,height=2)
        self.ageGroupsNumList.grid(row=7,column=2)
        self.labelCupComps = Label(self.mainScreenLeft, text='Cup Competitions:',
                                   bg=self.rgb_light_green).grid(row=1,column=3)       
        self.cupCompList = Listbox(self.mainScreenLeft, width=13,height=10)
        self.cupCompList.grid(row=2,column=3,padx=50,rowspan=7)
        self.populateListboxes()

        #Menubar - top of screen
        menubar = Menu(self.master)
        filemenu = Menu(menubar, tearoff=0)
        filemenu.add_command(label='Import Pupils',command=self.importPupils,accelerator="Ctrl+I")
        filemenu.add_command(label='User Manual',command=self.helpProg,accelerator="Ctrl+U")
        filemenu.add_command(label='Write Totals To File...',command=self.writeFile,accelerator="Ctrl+W")
        filemenu.add_command(label='Quit',command=self.quitProgram,accelerator="Ctrl+Q")
        screenmenu = Menu(menubar, tearoff=0)
        screenmenu.add_command(label='Results',command=self.showResultScreen,accelerator="Ctrl+R")
        screenmenu.add_command(label='Pupils',command=self.showPupilScreen,accelerator="Ctrl+P")
        screenmenu.add_command(label='Events',command=self.showEventScreen,accelerator="Ctrl+E")
        screenmenu.add_command(label='Manage Details',command=self.showManageDetailsScreen,accelerator="Ctrl+D")
        othersmenu = Menu(menubar, tearoff=0)
        othersmenu.add_command(label='Get Directory',command=self.getDirectory,accelerator="Ctrl+G")
        othersmenu.add_command(label='Reset',command=self.clearAllData,accelerator="Ctrl+S")
        menubar.add_cascade(label='File',menu=filemenu)
        menubar.add_cascade(label='Screens',menu=screenmenu)
        menubar.add_cascade(label='Others',menu=othersmenu)
        self.master.config(menu=menubar)


    def populateListboxes(self):
        #read in all current totals
        self.readInHouses()
        self.readInPupils()
        self.readInEvents()
        self.readInResults()
        self.readInCupComps()
        self.readInAgeGroups()
        self.houseTotal = 0
        self.pupilTotal = 0
        self.eventTotal = 0
        self.resultTotal = 0
        self.cupCompTotal = 0
        self.ageGroupTotal = 0
        for i in self.houseArray:
            self.houseTotal += 1
            self.houseList.insert(END, i)
        for i in self.allPupilTotal:
            self.pupilTotal += 1
        for i in self.eventArrayFullName:
            self.eventTotal += 1
        for i in self.resultArray:
            self.resultTotal += 1
        for i in self.cupCompArray:
            self.cupCompTotal += 1
            self.cupCompList.insert(END ,i)
        for i in self.ageGroupArray:
            self.ageGroupTotal += 1
        self.housesNumList.insert(END,self.houseTotal)
        self.pupilNumList.insert(END,self.pupilTotal)
        self.eventNumList.insert(END,self.eventTotal)
        self.resultNumList.insert(END,self.resultTotal)
        self.cupCompsNumList.insert(END,self.cupCompTotal)
        self.ageGroupsNumList.insert(END,self.ageGroupTotal)

        
    def refreshListboxes(self):
        #when 'refresh' button is pressed
        for i in self.houseArray:
            self.houseList.delete(END)
        self.housesNumList.delete(END)
        self.pupilNumList.delete(END)
        self.eventNumList.delete(END)
        self.resultNumList.delete(END)
        for i in self.cupCompArray:
            self.cupCompList.delete(END)
        self.cupCompsNumList.delete(END)
        self.ageGroupsNumList.delete(END)
        self.populateListboxes()


    #Display Window Procedures
    #Screen Procedures
    def showResultScreen(self):
        ResultScreenObject = Results()
        ResultScreenObject.configResultsScreen(self.master)
        
    def showPupilScreen(self):
        PupilScreenObject = Pupils()
        PupilScreenObject.configPupilsScreen(self.master)

    def showEventScreen(self):
        EventScreenObject = Events()
        EventScreenObject.configEventsScreen(self.master)

    def showManageDetailsScreen(self):
        ManageDetailsObject = ManageDetails()
        ManageDetailsObject.configManageDetailsScreen(self.master)

    def helpProg(self):
        myMessage = messagebox.askokcancel(title='Help',message='Press OK to open the user manual.')
        if myMessage == 1:
            os.startfile('userManual.pdf')


    def writeFile(self):
        #Import pupil csv
        myMessage = messagebox.askyesno(title='Confirm',message='''Would you like to write the final results to a file?''')
        if myMessage == 1:
            self.file = filedialog.askopenfile(mode='r',filetypes=[("CSV files","*.csv")])
            messageImported = messagebox.showinfo(title='Success',message='Totals have been completed.')



    def getDirectory(self):
        self.directory = os.path.dirname(os.path.realpath(__file__))
        direcMessage = messagebox.showinfo(title='Get Directory',message='Current directory: \n%s' %(self.directory))

    def quitProgram(self):
        myMessage = messagebox.askyesno(title='Exit Program',message='Are you sure you want to exit the program?')
        if myMessage == 1:
            self.master.destroy()




            
class Results(Toplevel,mainWindow):
    def __init__(self):
        #Open database
        database_opts.__init__(self)
        '''create arrays for use in the program, string variables created for use in entry boxes'''
        self.genderValues = ['Male','Female']
        self.eventTypes = ['Relay','Track','Field']
        self.orderArray = ['asc','des']
        self.resultEntry = StringVar()
        self.pupilNames = []
        self.eventNameShow = StringVar()
        self.surnameSearch = StringVar()
        self.eventNames = []
        self.cupCompVal = StringVar()
        self.currentAgeGroup = StringVar()

        
    def configResultsScreen(self,master):
        #Make current window 'TopLevel'
        Toplevel.__init__(self)
        #Read in values from database
        self.readInHouses()
        self.readInPupils()
        self.readInEvents()
        self.readInAgeGroups()
        self.readInCupComps()

        #Define Results Window
        self.master = master
        self.title('Result Screen')
        self.geometry('1000x590+120+30')
        self.master.configure(background=self.rgb_light_green)
        self.resizable(0,0)
        self.eventNameShow.set('No Event Selected')
        #Notebook widget on window
        self.note = ttk.Notebook(self, width=1000,height=590)
        self.addResultTab = Frame(self.note,bg=self.rgb_light_green)
        self.totalsTab = Frame(self.note,bg=self.rgb_light_green)
        self.championsTab = Frame(self.note,bg=self.rgb_light_green)
        #Add tabs to notebook widget
        self.note.add(self.addResultTab, text='Add Result')
        self.note.add(self.totalsTab, text='Totals')
        self.note.add(self.championsTab, text='Champions')
        self.note.pack()


        #View/Add Results Window
        #Frames
        self.arfTopResult = Frame(self.addResultTab,pady=10,bg=self.rgb_light_green)
        self.arfTopResult.pack(side=TOP)
        self.arfLeftResult = Frame(self.addResultTab,pady=30,padx=10,bg=self.rgb_light_green)
        self.arfLeftResult.pack(side=LEFT)
        self.arfRightResult = Frame(self.addResultTab,pady=30,padx=40,bg=self.rgb_light_green)
        self.arfRightResult.pack(side=RIGHT)

        #Top frame - title for add results
        self.labelMain = Label(self.arfTopResult, text='Add Results',font=('Courier',24),
                               bg=self.rgb_light_green,relief=RAISED).grid(row=0,column=0)

        #Left frame - search for events to view results for and provide widgets for result entry
        self.eventNamesLabelAR = Label(self.arfLeftResult, text='Select an Event:',
                                       bg=self.rgb_light_green).grid(row=0,column=1,pady=10)
        self.eventTypeLabelAR = Label(self.arfLeftResult,text='Event Type?',
                                      bg=self.rgb_light_green).grid(row=1,column=0,pady=2)
        self.ageGroupLabelAR = Label(self.arfLeftResult,text='Age Group?',
                                     bg=self.rgb_light_green).grid(row=2,column=0,pady=5)
        self.eventFindBut = Button(self.arfLeftResult,text='Find',command=self.findEvents,
                                   highlightbackground=self.rgb_light_green).grid(row=3,column=1)
        self.eventNamesListyLabelAR = Label(self.arfLeftResult, text='Event: ',
                                            bg=self.rgb_light_green).grid(row=4,column=0)
        self.showCurrentButton = Button(self.arfLeftResult, text='Show Current Results ',
                                        highlightbackground=self.rgb_light_green,
                                        command=self.checkEventSelection).grid(row=5,column=1,pady=(0,5))
        self.pupilDetailsLabelAR = Label(self.arfLeftResult, text='Find Pupils:',
                                         bg=self.rgb_light_green).grid(row=6,column=1,pady=10)
        self.searchSurnameLabelAR = Label(self.arfLeftResult, text='Surname: ',
                                          bg=self.rgb_light_green).grid(row=7,column=0,pady=10)
        self.surnameEntryAR = Entry(self.arfLeftResult, textvariable=self.surnameSearch).grid(row=7,column=1)
        self.applySearchButtonAR = Button(self.arfLeftResult, text='Search',
                                          highlightbackground=self.rgb_light_green,
                                        command=self.applySearchPupilR).grid(row=8,column=1,pady=(0,20))
        self.pupilNameLabelAR = Label(self.arfLeftResult, text='Pupil Name: ',
                                      bg=self.rgb_light_green).grid(row=9,column=0)
        self.timeDistanceLabelAR = Label(self.arfLeftResult, text='Time/Distance: ',
                                         bg=self.rgb_light_green).grid(row=10,column=0)
        self.resultEntryboxAR = Entry(self.arfLeftResult, textvariable=self.resultEntry).grid(row=10,column=1)
        self.addResultButtonAR = Button(self.arfLeftResult, text='Add Result', command=self.addResult,
                                        highlightbackground=self.rgb_light_green).grid(row=12,column=1,pady=(5,0))

        #Right frame - labels, listboxes to display results and button widgets
        self.eventNameLabelAR = Label(self.arfRightResult, textvariable=self.eventNameShow,bg=self.rgb_light_green).grid(row=0,column=0,
                                                                                                                         columnspan=4,pady=10)
        self.pupilListLabelAR = Label(self.arfRightResult, text='Pupil: ',bg=self.rgb_light_green).grid(row=2,column=0)
        self.pupilListyAR = Listbox(self.arfRightResult, width=14, height=12)
        self.pupilListyAR.grid(row=4,column=0)
        self.houseNameLabelAR = Label(self.arfRightResult, text='House: ',bg=self.rgb_light_green).grid(row=2,column=1)
        self.houseNameListyAR = Listbox(self.arfRightResult, width=8, height=12)
        self.houseNameListyAR.grid(row=4,column=1)
        self.resultValLabelAR = Label(self.arfRightResult, text='Result: ',bg=self.rgb_light_green).grid(row=2,column=2)
        self.resultValListyAR = Listbox(self.arfRightResult, width=8, height=12)
        self.resultValListyAR.grid(row=4,column=2)
        self.pointsValLabelAR = Label(self.arfRightResult, text='Points: ',bg=self.rgb_light_green).grid(row=2,column=3)
        self.pointsValListyAR = Listbox(self.arfRightResult, width=8, height=12)
        self.pointsValListyAR.grid(row=4,column=3)
        self.guestsLabelAR = Label(self.arfRightResult, text='Guests: ',bg=self.rgb_light_green).grid(row=2,column=4,padx=(20,0))
        self.guestsValListyAR = Listbox(self.arfRightResult, width=13, height=12)
        self.guestsValListyAR.grid(row=4,column=4,padx=(20,0))
        self.guestsLabelPointAR = Label(self.arfRightResult, text='Result: ',bg=self.rgb_light_green).grid(row=2,column=5)
        self.guestsResultValListyAR = Listbox(self.arfRightResult, width=8, height=12)
        self.guestsResultValListyAR.grid(row=4,column=5)
        self.deleteResultButtonAR = Button(self.arfRightResult, text='Delete Result', command=self.deleteResult,
                                           highlightbackground=self.rgb_light_green).grid(row=7,column=0,pady=20)
        self.deleteResultGButtonAR = Button(self.arfRightResult, text='Delete Result', command=self.deleteGuestResult,
                                            highlightbackground=self.rgb_light_green).grid(row=7,column=4,pady=20,columnspan=2)


        #Totals Window
        #Frames
        self.tpfTopTotals = Frame(self.totalsTab,padx=30,pady=10,bg=self.rgb_light_green)
        self.tpfTopTotals.pack(side=TOP)
        self.tpfLeftTotals = Frame(self.totalsTab,padx=80,pady=20,bg=self.rgb_light_green)
        self.tpfLeftTotals.pack(side=LEFT)

        #Top Frame - title for totals
        self.labelHouseResults = Label(self.tpfTopTotals, text='Totals',font=('Courier',24),
                                       bg=self.rgb_light_green,relief=RAISED).grid(row=0,column=0)
        
        #Left frame - search for totals by cup competition or age group and display house totals
        self.currentCTotalsLabel = Label(self.tpfLeftTotals, text='Cup Competition Totals:',bg=self.rgb_light_green).grid(row=0,column=0,
                                                                                                                          pady=(0,10),columnspan=2)
        self.cupCompFindBut = Button(self.tpfLeftTotals,text='Find',highlightbackground=self.rgb_light_green,
                                     command=self.findCompTotals).grid(row=2,column=0,columnspan=2)
        self.cupCompVal.set('No Competition Selected')
        self.currentCupComp = Label(self.tpfLeftTotals,textvariable=self.cupCompVal,bg=self.rgb_light_green).grid(row=4,column=0,columnspan=2,
                                                                                                                  pady=(30,10))
        self.houseNamesListyLabel = Label(self.tpfLeftTotals, text='House Name:',bg=self.rgb_light_green).grid(row=5,column=0)
        self.cupHouseListy = Listbox(self.tpfLeftTotals, width=15, height=10)
        self.cupHouseListy.grid(row=6,column=0)
        self.cupTotalsListyLabel = Label(self.tpfLeftTotals, text='Totals:',bg=self.rgb_light_green).grid(row=5,column=1)
        self.cupTotalsListy = Listbox(self.tpfLeftTotals, width=15,height=10)
        self.cupTotalsListy.grid(row=6,column=1)
        self.currentATotalsLabel = Label(self.tpfLeftTotals, text='Age Group Totals:',bg=self.rgb_light_green).grid(row=0,column=4,
                                                                                                                    pady=(0,10),columnspan=2,padx=(30,0))
        self.ageGroupFindBut = Button(self.tpfLeftTotals,text='Find',highlightbackground=self.rgb_light_green,
                                      command=self.findAgeTotals).grid(row=2,column=4,columnspan=2,padx=(30,0))
        self.currentAgeGroup.set('No Age Group Selected')
        self.currentAgeGroups = Label(self.tpfLeftTotals,textvariable=self.currentAgeGroup,bg=self.rgb_light_green).grid(row=4,column=4,
                                                                                                                         columnspan=2,pady=(30,10),
                                                                                                                         padx=(30,0))
        self.ageHouseLabel = Label(self.tpfLeftTotals, text='House Name:',bg=self.rgb_light_green).grid(row=5,column=4,padx=(30,0))
        self.ageHouseListy = Listbox(self.tpfLeftTotals, width=15, height=10)
        self.ageHouseListy.grid(row=6,column=4,padx=(30,0))
        self.ageTotalsListyLabel = Label(self.tpfLeftTotals, text='Totals:',bg=self.rgb_light_green).grid(row=5,column=5)
        self.ageTotalsListy = Listbox(self.tpfLeftTotals, width=15,height=10)
        self.ageTotalsListy.grid(row=6,column=5)
        self.currentHTotalsLabel = Label(self.tpfLeftTotals, text='Current House Totals:',bg=self.rgb_light_green).grid(row=4,column=8,
                                                                                                                        columnspan=2,pady=(30,10))
        self.houseNamesListyLabel = Label(self.tpfLeftTotals, text='House Name:',bg=self.rgb_light_green).grid(row=5,column=8,padx=(30,0))
        self.houseNamesListy = Listbox(self.tpfLeftTotals, width=15, height=10)
        self.houseNamesListy.grid(row=6,column=8,padx=(30,0))
        self.houseTotalsListyLabel = Label(self.tpfLeftTotals, text='Totals:',bg=self.rgb_light_green).grid(row=5,column=9)
        self.houseTotalsListy = Listbox(self.tpfLeftTotals, width=15,height=10)
        self.houseTotalsListy.grid(row=6,column=9)

        
        #Champions Window
        #Frames
        self.crfTopChamp = Frame(self.championsTab,pady=10,bg=self.rgb_light_green)
        self.crfTopChamp.pack(side=TOP)
        self.crfLeftChamp = Frame(self.championsTab,padx=30,pady=10,bg=self.rgb_light_green)
        self.crfLeftChamp.pack()
        self.crfBottomChamp = Frame(self.championsTab,padx=30,pady=10,bg=self.rgb_light_green)
        self.crfBottomChamp.pack()

        #Top frame - title for champions
        self.labelChampions = Label(self.crfTopChamp, text='Champions',font=('Courier',24),
                                    bg=self.rgb_light_green,relief=RAISED).grid(row=0,column=1)

        #Left frame - search for champions by age group, gender and number of champions using the spinbox 
        self.indivChampLabel = Label(self.crfLeftChamp, text='Search For Champions:',bg=self.rgb_light_green).grid(row=1,column=1,pady=10)
        self.indivNameLabel = Label(self.crfLeftChamp, text='Select an Age Group:',bg=self.rgb_light_green).grid(row=2,column=0)
        self.genderChampLabel = Label(self.crfLeftChamp, text='Select a Gender:',bg=self.rgb_light_green).grid(row=3,column=0)
        self.numChampsLabel = Label(self.crfLeftChamp, text='No. of Champions:',bg=self.rgb_light_green).grid(row=4,column=0)
        self.numChampBox = Spinbox(self.crfLeftChamp, from_=0, to=10, highlightbackground=self.rgb_light_green, state='readonly')
        self.numChampBox.grid(row=4,column=1,pady=10)
        self.applyChampButton = Button(self.crfLeftChamp, text='Find',command=self.populateChampionsListbox,
                                       highlightbackground=self.rgb_light_green).grid(row=5,column=1,pady=10)

        #Bottom frame - display the found champions in the listboxes
        self.championsNameLabel = Label(self.crfBottomChamp, text='Pupil Name:',bg=self.rgb_light_green).grid(row=0,column=0)
        self.championsNameListbox = Listbox(self.crfBottomChamp, width=18,height=10)
        self.championsNameListbox.grid(row=1,column=0,pady=8)
        self.championsHouseLabel = Label(self.crfBottomChamp, text='House:',bg=self.rgb_light_green).grid(row=0,column=1)
        self.championsHouseListbox = Listbox(self.crfBottomChamp, width=13,height=10)
        self.championsHouseListbox.grid(row=1,column=1,pady=8)
        self.championsPointsLabel = Label(self.crfBottomChamp, text='Total Points:',bg=self.rgb_light_green).grid(row=0,column=2)
        self.championsPointsListbox = Listbox(self.crfBottomChamp, width=11,height=10)
        self.championsPointsListbox.grid(row=1,column=2,pady=8)

        #combobox methods, refresh pupil search variables and calculate house totals
        self.eventTypeCombo()
        self.eventCombo()
        self.ageGroupCombo()
        self.genderCombo()
        self.cupCompCombo()
        self.initPupSearch()
        self.findHouseTotals()

        
    #Comboboxes
    def eventTypeCombo(self):
        #add/view results event type combobox
        self.eventTCombo_valueSearch = StringVar()
        self.eventTComboboxSearch = ttk.Combobox(self.arfLeftResult,
                                                 textvariable=self.eventTCombo_valueSearch,width=20,state='readonly')
        self.eventTComboboxSearch['values'] = (self.eventTypes)
        self.eventTComboboxSearch.grid(row=1,column=1)
        
    def eventCombo(self):
        #add/view results events combobox
        self.eventCombo_valuePRSearch = StringVar()
        self.eventComboboxPRSearch = ttk.Combobox(self.arfLeftResult,
                                                  textvariable=self.eventCombo_valuePRSearch,width=30,state='readonly')
        self.eventComboboxPRSearch['values'] = (self.eventNames)
        self.eventComboboxPRSearch.grid(row=4,column=1,pady=10)
        
    def ageGroupCombo(self):
        #add/view results age group combobox
        self.ageGroup_valuePRSearch = StringVar()
        self.ageGroupComboboxPRSearch = ttk.Combobox(self.arfLeftResult,
                                                     textvariable=self.ageGroup_valuePRSearch,state='readonly')
        self.ageGroupComboboxPRSearch['values'] = (self.ageGroupArray)
        self.ageGroupComboboxPRSearch.grid(row=2,column=1)
        #champions age group combobox
        self.ageGroup_valueCSearch = StringVar()
        self.ageGroupComboboxCSearch = ttk.Combobox(self.crfLeftChamp,
                                                    textvariable=self.ageGroup_valueCSearch,state='readonly')
        self.ageGroupComboboxCSearch['values'] = (self.ageGroupArray)
        self.ageGroupComboboxCSearch.grid(row=2,column=1,pady=5)
        #totals age group combobox
        self.ageGroup_valueTSearch = StringVar()
        self.ageGroupComboboxTSearch = ttk.Combobox(self.tpfLeftTotals,
                                                    textvariable=self.ageGroup_valueTSearch,state='readonly')
        self.ageGroupComboboxTSearch['values'] = (self.ageGroupArray)
        self.ageGroupComboboxTSearch.grid(row=1,column=4,pady=5,columnspan=2,padx=(30,0))
               
    def genderCombo(self):
        #champions gender combobox
        self.gender_valueCSearch = StringVar()
        self.genderComboboxCSearch = ttk.Combobox(self.crfLeftChamp,
                                                  textvariable=self.gender_valueCSearch,state='readonly')
        self.genderComboboxCSearch['values'] = (self.genderValues)
        self.genderComboboxCSearch.grid(row=3,column=1)

    def pupilSearchResultCombo(self):
        #add/view results result combobox
        self.pupilSearchR_value = StringVar()
        self.pupilComboboxPSearch = ttk.Combobox(self.arfLeftResult,
                                                 textvariable=self.pupilSearchR_value,state='readonly')
        self.pupilComboboxPSearch['values'] = (self.pupilNames)
        self.pupilComboboxPSearch.grid(row=9,column=1)

    def cupCompCombo(self):
        #totals cup competition combobox
        self.cupCompSearchR_value = StringVar()
        self.cupCompSearch = ttk.Combobox(self.tpfLeftTotals,
                                          textvariable=self.cupCompSearchR_value,state='readonly')
        self.cupCompSearch['values'] = (self.cupCompArray)
        self.cupCompSearch.grid(row=1,column=0,columnspan=2)           

    '''This method initialises all the variables and arrays needed for searching for pupils'''           
    def initPupSearch(self):
        #pupil search variables
        self.pupilNames = []
        self.pupSearchVals = []
        self.pupilSearchDetails = []
        self.pupilSearchResultCombo()
        #champions
        self.championsArray = []
        self.championsValues = []


    '''This method allows the user to add a result into the database. Validation checks are also used to ensure
       the data entered is 'correct' and if not, display relevant error messages. Once a result has been added,
       the entry boxes and listboxes will reset and display the current results for the selected event'''
    def addResult(self):
        #ensure user has entered a result to add
        if self.resultEntry.get() == '':
            errormessage= messagebox.showwarning(title='Error',message='Please ensure you have entered a result.')
        else:
            if self.pupilSearchR_value.get() == '' or self.eventCombo_valuePRSearch.get() == '':
                errormessage= messagebox.showwarning(title='Error',
                                                     message='Please ensure you have selected an event and searched for a pupil.')
            else:
                #get the selected event and pupil to add the result for
                self.repeatVal = 0
                self.eventToSearch = self.eventCombo_valuePRSearch.get()
                self.currentPupilIndex = self.pupilComboboxPSearch.current()
                self.pupilIDSelect = self.pupilSearchDetails[self.currentPupilIndex]
                self.pupilIDSelectEdit = self.pupilIDSelect[0]
                #validate the result entry
                self.validateResultEntry()
                if self.failed == True:
                    errormessage = messagebox.showwarning(title='Error',
                                                          message='Please ensure you have entered a number.')
                else:
                    #check for an existing result
                    self.checkForRepeatResults()
                    if self.repeatVal == 1:
                        errormessage = messagebox.showwarning(title='Error',
                                                              message='Pupil already has a result for this event.')                      
                        #reset entry boxes
                        self.surnameSearch.set('')
                        self.pupilSearchR_value.set('')
                        self.resultEntry.set('')
                    else:
                        #allows user to confirm their input
                        myMessage = messagebox.askyesno(title='Add Result',
                                                        message='Result: %s \nIs this correct?' %(self.resultEntry.get()))
                        if myMessage == 1:
                            #add the result
                            self.insertResult(self.resultEntry.get(),self.pupilIDSelectEdit,self.eventCombo_valuePRSearch.get())
                            #reset entry boxes
                            self.surnameSearch.set('')
                            self.pupilSearchR_value.set('')
                            self.resultEntry.set('')
                            #re-calculate the house totals
                            self.findHouseTotals()
                            #reset the listboxes
                            self.refreshListboxes()


    '''This method validates the user's entry to ensure the result is a float'''              
    def validateResultEntry(self):
        #ensure result is float
        try:
            float(self.resultEntry.get())
            self.failed = False
        except ValueError:
            self.failed = True
            

    '''This method checks for an existing result in the database. If their is a result that already exists
       for a certain pupil, the variable repeatVal will be set to 1 and an error message will be displayed'''
    def checkForRepeatResults(self):
        #self.repeatVal = 1 if repeat result is present
        self.indivCheckPupil = []
        self.checkRepeats()
        for i in self.pupilCheck:
            self.indivCheckPupil.append(i)
        for i in self.indivCheckPupil:
            if i[0] == self.pupilIDSelect[0] and i[4] == self.eventToSearch:
                self.repeatVal = 1


    '''This method checks for a selected event in the event combobox on the add/view results window'''
    def checkEventSelection(self):
        self.eventToSearch = self.eventCombo_valuePRSearch.get()
        #display an error id the user hasn't selected an event from the combobox
        if self.eventToSearch == '':
            errormessage = messagebox.showwarning(title='Error',message='Please select an event.')
        else:
            self.showCurrentResults()

        
    '''This method takes the event selected on add/view results and finds all the results that correspond
       to that event. The listboxes will reset and the points will be calculated for all the results found from
       the database'''
    def showCurrentResults(self):
        #displays the list of results for the chosen event
        self.eventToSearch = self.eventCombo_valuePRSearch.get()
        if self.eventToSearch == '':
            pass
        else:
            self.eventNameShow.set(self.eventToSearch)
            self.eventAndResultType(self.eventToSearch)
            self.refreshListboxesR()
            self.calcPoints(self.eventToSearch)


    '''This method takes the selected cup competition on the Totals window and calculates the total points
       for each house for that cup compeititon selection'''
    def findCompTotals(self):
        if self.cupCompSearchR_value.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='Please select a cup competition.')
        else:
            #contains all houses with a '0' to start
            self.masterHousePoints = []
            for house in self.houses:
                self.masterHousePoints.append(0)
            #reset cup competition listbox
            self.cupHouseListy.delete(0,END)
            #insert all the houses into the houses listbox
            for i in self.houseArray:
                self.cupHouseListy.insert(END,i)
            #displays to the user their cup competition selection
            self.cupCompVal.set(self.cupCompSearchR_value.get() + ' ' + 'Totals:')
            
            #finds all the events by the selected cup competition
            self.cur.execute("SELECT * FROM Event WHERE Event.compName=?", (self.cupCompSearchR_value.get(),))                        
            self.allEvents = self.cur.fetchall()
            #re-calculate totals
            self.calcTotals()

            #reset the totals listbox for the cup competitions
            for i in self.houseArray:
                self.cupTotalsListy.delete(END)
            #fill the totals listbox with the totals for the cup competition
            for p in self.masterHousePoints:
                self.cupTotalsListy.insert(END, p)
                
        #refresh listboxes on add/view results window
        self.showCurrentResults()


    '''This method takes the selected age group on the Totals window and calculates the total points
       for each house for that age group selection'''
    def findAgeTotals(self):
        if self.ageGroup_valueTSearch.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='Please select an age group.')
        else:
            #contains all houses with a '0' to start
            self.masterHousePoints = []
            for house in self.houses:
                self.masterHousePoints.append(0)
            #reset age group listbox
            self.ageHouseListy.delete(0,END)
            #insert all the houses into the houses listbox
            for i in self.houseArray:
                self.ageHouseListy.insert(END,i)
            #displays to the user their age group selection
            self.currentAgeGroup.set(self.ageGroup_valueTSearch.get() + ' ' + 'Totals:')
            
            #finds any event by the selected age group
            self.findAllAgeFromEvents(self.ageGroup_valueTSearch.get())
            #re-calculate totals
            self.calcTotals()

            #reset the totals listbox for the age groups
            for i in self.houseArray:
                self.ageTotalsListy.delete(END)
            #fill the totals listbox with the totals for the age group
            for p in self.masterHousePoints:
                self.ageTotalsListy.insert(END, p)

        #refresh listboxes on Totals window
        self.showCurrentResults()


    '''This method calculates the total points for each house using all the age groups from the database.
       The totals for each house will update in the listboxes on the Totals window if a result is added
       or deleted'''
    def findHouseTotals(self):
        #contains all houses with a '0' to start
        self.masterHousePoints = []
        for house in self.houses:
            self.masterHousePoints.append(0)
        #reset house names listbox
        self.houseNamesListy.delete(0,END)
        #insert all the houses into the houses listbox
        for i in self.houseArray:
            self.houseNamesListy.insert(END,i)
        #uses all the age groups and searches for all events by that age group
        for age in self.ageGroupArray:
            self.findAllAgeFromEvents(age)
            #re-calculate all the totals for each house from each age group
            self.calcTotals()
        #reset the totals listbox for the houses
        for i in self.houseArray:
            self.houseTotalsListy.delete(END)
        #fill the totals listbox with the totals for each house
        for p in self.masterHousePoints:
            self.houseTotalsListy.insert(END, p)


    '''This method is called each time the total points need to be updated. All the events are found
       and for each event, the array scoreResults will contain the list of pupils who scored for that
       event. For each of these pupils points, update the pupil's house totals for the amount of points
       they scored'''
    def calcTotals(self):
        for event in self.allEvents:
            #find all the results from all the events in the database
            self.eventAndResultType(event[1] + ' ' + event[2] + ' ' + event[3])
            self.resultsByEvent(event[1] + ' ' + event[2] + ' ' + event[3])
            #generates scoreResults and pointResults
            self.sortScorers()
            #for each position in scoreResults, result will equal the position in scoreResults
            for position in range(len(self.scoreResults)):
                result = self.scoreResults[position]
                #if the pupil is ineligible or there is no points for that event, ignore
                if result[3] == -1 or len(self.pointResults) == 0:
                    pass
                else:
                    #get the house that corresponds to the scoring pupil
                    houseindex = self.houseArray.index(result[3])
                    #update that position in masterHousePoints with the points the pupil scored for that event
                    self.masterHousePoints[houseindex] += self.pointResults[position]

                        
    '''This method is used to calculate the points scored for each eligible pupil in an event. It takes
       the paramater eventToSearch, so for every event, find the results and generate a list of scoreResults
       and pointResults for that event. Once these lists have been filled, update the listboxes on add/view
       results to display to the user'''
    def calcPoints(self, eventToSearch):
        #find all the results for the event
        self.resultsByEvent(eventToSearch)
        #ignore the event if there is no results
        if len(self.results) == 0:
            pass
        else:
            #generate a list of scoreResults and pointResults for that event
            self.sortScorers()
            #at this point, pointResults and scoreResults has all sorted points and pupils for an event
            #if the result type is seconds, fill the listboxes with the scoring pupils
            if self.eAndRTypeArray[1] == 'seconds':
                for i in range(self.numPupils):
                    #insert position and name
                    self.pupilListyAR.insert(END, str(i+1) + '.' + ' ' + self.scoreResults[i][1][0] + ' ' + self.scoreResults[i][2])
                    #insert house name
                    self.houseNameListyAR.insert(END, self.scoreResults[i][3][0])
                    #insert the pupils result
                    self.resultValListyAR.insert(END, str(self.scoreResults[i][4]) + '' + str(self.eAndRTypeArray[1][0]))
                for i in self.pointResults:
                    #insert the points the pupil scored for that event
                    self.pointsValListyAR.insert(END, i)
                for i in self.guestArray:
                    #insert the guests name (didn't finish in the top 8 andt top 2 scoring results from their house
                    self.guestsValListyAR.insert(END, i[1][0] + ' ' + i[2])
                    #insert the guests results
                    self.guestsResultValListyAR.insert(END, str(i[4]) + '' + str(self.eAndRTypeArray[1][0]))
            else:
                #if the result type isn'nt seconds, fill the listboxes with the scoring pupils
                for i in range(self.numPupils):
                    #insert position and name
                    self.pupilListyAR.insert(END, str(i+1) + '.' + ' ' + self.scoreResults[i][1][0] + ' ' + self.scoreResults[i][2])
                    #insert house name
                    self.houseNameListyAR.insert(END, self.scoreResults[i][3][0])
                    #insert the pupils result
                    self.resultValListyAR.insert(END, str(self.scoreResults[i][4]) + '' + str(self.eAndRTypeArray[1][0]))
                for i in self.pointResults:
                    #insert the points the pupil scored for that event
                    self.pointsValListyAR.insert(END, i)
                for i in self.guestArray:
                    #insert the guests name (didn't finish in the top 8 andt top 2 scoring results from their house
                    self.guestsValListyAR.insert(END, i[1][0] + ' ' + i[2])
                    #insert the guests results
                    self.guestsResultValListyAR.insert(END, str(i[4]) + '' + str(self.eAndRTypeArray[1][0]))


    '''This method is used to generate 2 lists - scoreResults and pointResults. The list scoreResults will contain
       the elgible pupils who scored for that event (finished in the top 2 scoring results for their house and finished
       within the top 8 scorers for the event. The points can then be assigned to the scorers in assignPoints'''
    def sortScorers(self):
        #list for socring pupils
        self.scoreResults = []
        #list for pupils points
        self.pointResults = []
        #list for guests
        self.guestArray = []
        #sort the list of guests result in descending order
        self.sortList(self.guestArray,self.orderArray[1])
        #sort the list by ascending if the event type is track or relay
        if self.eAndRTypeArray[0] == 'Track' or self.eAndRTypeArray[0] == 'Relay':
            self.sortList(self.results,self.orderArray[0])
        else:
            #sort the list by descending if not track or relay
            self.sortList(self.results,self.orderArray[1])

        #for each result, if that result matches to a house, append to houseresult
        #for each value in houseresult, if the number of houses is less than 2 append to scoreResults
        #otherwise pupil is a 'guest'                  
        for house in self.houses:
            self.houseresult = []
            for result in self.results:
                if result[3][0] == house:
                    self.houseresult.append(result)
            self.addCount = 0   
            for r in self.houseresult:
                if self.addCount < 2:
                    self.scoreResults.append(r)
                    self.addCount +=1
                else:
                    self.guestArray.append(r)

        #sort the list by ascending if the event type is track or relay
        if self.eAndRTypeArray[0] == 'Track' or self.eAndRTypeArray[0] == 'Relay':
            self.sortList(self.scoreResults,self.orderArray[0])
        else:
            #sort the list by descending if the event type isn't track or relay
            self.sortList(self.scoreResults,self.orderArray[1])

        #if there is at least 1 scorer for an event, assign the points to the scorers
        if len(self.scoreResults)>0:
            self.assignPoints(self.pointResults,self.scoreResults)
        #number of scorers
        self.numPupils = len(self.pointResults)



    '''This method is used to assign points to the scorers for an event (scoreResults) and stores in pointResults.
       It also checks for the same result and will calculate an average if necessary'''
    def assignPoints(self,pointResults,scoreResults):
        #points for allocation (not index[0] or index[9])
        self.pointList = [0,9,7,6,5,4,3,2,1,0]
        self.equalCount = 1
        self.startEqual = 0
        self.shareresult = self.pointList[1]
        self.blankResult = [-1,'','',-1,1]
        self.scoreResults.append(self.blankResult)
        #for each position in scoreResults
        for pos in range(0,len(self.scoreResults)-1):
            #if the result for that pupil equals the pupils result after
            if self.scoreResults[pos][4] == self.scoreResults[pos+1][4]:
                #increment equalCount
                self.equalCount +=1
                #set the shared result
                self.shareresult += self.pointList[pos+2]
            else:
                #if there is a shared result
                if self.equalCount !=1:
                    #calculate an average of both points and the number of matching results to 1.d.p
                    self.pointsEach = round((self.shareresult/(self.equalCount)), 1)
                    for ppos in range (self.startEqual,pos+1):
                        #append the new average points to pointResults
                        self.pointResults.append(self.pointsEach)
                    #set equalCount back to 1
                    self.equalCount = 1
                    self.startEqual = pos+1
                    self.shareresult = self.pointList[pos+2]
                else:
                    #if there is not a shared result, append the points to pointResult
                    self.pointResults.append(self.shareresult) 
                    self.shareresult = self.pointList[pos+2]
                    self.startEqual = pos+1


    '''This method sorts a list in a certain order depending on the value in 'order' using a simple bubblesort
       algorithm'''
    def sortList(self, listy, order):
        #order list in ascending or descending order
        sorted = False
        while not sorted:
            sorted = True
            for result in range(0,len(listy)-1):
                #if the order is desceding, bubblesort the list in descending order
                if order == 'des':
                    if listy[result][4] < listy[result+1][4]:
                        temp = listy[result]
                        listy[result] = listy[result+1]
                        listy[result+1] = temp
                        sorted = False
                else:
                    #bubblesort the list in ascending order
                    if listy[result][4] > listy[result+1][4]:
                        temp = listy[result]
                        listy[result] = listy[result+1]
                        listy[result+1] = temp
                        sorted = False
                        

    '''This method resets the listboxes on the add/view results window and then displays the results from the database
       for the selected event once showCurrentResults is called'''
    def refreshListboxes(self):
        #delete all values from listboxes and show current results
        for i in self.results:
            self.pupilListyAR.delete(END)
            self.houseNameListyAR.delete(END)
            self.resultValListyAR.delete(END)
        for i in self.results:
            self.pointsValListyAR.delete(END)
        self.showCurrentResults()


    '''This method resets the listboxes on the add/view results window'''
    def refreshListboxesR(self):
        #clear all values from all listboxes
        self.pupilListyAR.delete(0,END)
        self.houseNameListyAR.delete(0,END)
        self.resultValListyAR.delete(0,END)
        self.pointsValListyAR.delete(0,END)
        self.guestsValListyAR.delete(0,END)
        self.guestsResultValListyAR.delete(0,END)


    '''This method allows the user to delete a result from the add/view results window's listboxes once a pupil
       has been selected and clicked 'Delete Result', which will also update the database to match their changes'''
    def deleteResult(self):
        #displays an error message if the user hasn't selected a pupil
        if self.pupilListyAR.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='No pupil selected to delete.')
        else:
            #re-calculate the scorers for the selected event
            self.sortScorers()
            #confirmation message for the user
            myMessage = messagebox.askyesno(title='Delete Result',message='Are you sure you want to delete the result for %s?'
                                            %(self.pupilListyAR.get(ACTIVE)))
            if myMessage == 1:
                #finds the selected pupil's ID
                self.pupilFindDel = (self.scoreResults[int(self.pupilListyAR.curselection()[0])])[0]
                #gets the current event selection
                self.currentEventDel = self.eventCombo_valuePRSearch.get()
                #removes the result from the database using the pupil's ID and the event selection
                self.removeResult(self.pupilFindDel,self.currentEventDel)
                #reset the listboxes on add/view results
                self.refreshListboxes()
                #re-calculate the house totals
                self.findHouseTotals()


    '''This method allows the user to delete a guest's result from the add/view results window's listboxes once a
       guest has been selected and clicked 'Delete Result', which will also update the database to match their changes'''
    def deleteGuestResult(self):
        #displays an error message if the user hasn't selected a guest
        if self.guestsValListyAR.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='No guest selected to delete.')
        else:
            #confirmation message for the user
            myMessage = messagebox.askyesno(title='Delete Guest Result',message='Are you sure you want to delete the result for %s?'
                                            %(self.guestsValListyAR.get(ACTIVE)))
            if myMessage == 1:
                #finds the selected pupil's ID
                self.pupilFindDel = (self.guestArray[int(self.guestsValListyAR.curselection()[0])])[0]
                #gets the current event selection
                self.currentEventDel = self.eventCombo_valuePRSearch.get()
                #removes the result from the database using the pupil's ID and the event selection
                self.removeResult(self.pupilFindDel,self.currentEventDel)
                #reset the listboxes on add/view results
                self.refreshListboxes()
                #re-calculate the house totals
                self.findHouseTotals()

    
    '''This method searches for pupils to populate the pupils combobox on the add/view results window. Once the user
       has entered a surname to search in the surname entry box and clicked 'Search', the pupils combobox will display
       all the pupils with that surname'''
    def applySearchPupilR(self):
        #gets the surname entry and adds to surnameArray
        self.surnameArray = []
        self.surnameArray.append(self.surnameSearch.get())
        #capitalize the first letter of the surnames in surnameArray
        for i in range(len(self.surnameArray)):
            self.surnameArray[i]=self.surnameArray[i].capitalize()

        
        #display an error message if the user hasn't entered a surname
        if self.surnameSearch.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='Please ensure you have entered a surname to search.')
        else:
            #the age group and gender to search will match the age group and gender for the selected event
            self.ageGroupSearchR = self.ageGroup_valuePRSearch.get()
            self.genderSearchR = self.eventArraySeparate[self.eventComboboxPRSearch.current()][3]

            #initialise all the relevant variables for searching
            self.initPupSearch()
            #checks the surname entry is text only 
            try:
                int(self.surnameSearch.get())
                self.failed = True
            except ValueError:
                self.failed = False
            #display an error message if the entry isn't text
            if self.failed == True:
                errormessage = messagebox.showwarning(title='Error',message='Please only use text for your search.')
            else:
                #confirmation message
                myMessage = messagebox.askyesno(title='Search Pupil',
                                                message='''You are searching for a pupil with details:
                                                \n \n Surname:  %s \n Age Group:  %s \n Gender:  %s \n \n Is this correct?'''
                                                %(self.surnameArray[0],self.ageGroupSearchR,self.genderSearchR))
                if myMessage == 1:
                    #search for pupils based on the surname, age group and gender
                    self.searchForPupilR(self.surnameArray[0],self.ageGroupSearchR,self.genderSearchR)
                    for i in self.pupSearchVals:
                        #for each pupil found, append to the array which is displayed in the combobox
                        self.pupilNames.append(i[1] + ' ' + i[2])
                        self.pupilSearchDetails.append(i)
                    #call the pupil combobox method to update                       
                    self.pupilSearchResultCombo()
                    #if no pupils are found, display an error message and reset the surname entry
                    if len(self.pupSearchVals) < 1:
                        errormessage = messagebox.showwarning(title='Error',message='No pupils found.')
                        self.surnameSearch.set('')


    '''This method finds all the events based on the selected event type and age group the user has selected. The
       events combobox will display all the events found based on this search'''
    def findEvents(self):
        #display an error message if the user hasn't selected an event type or an age group
        if self.eventTCombo_valueSearch.get() == '' or self.ageGroup_valuePRSearch.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='Please select an event type and an age group to find events.')
        else:
            self.eventsFoundByCriteria = []
            #find all the events that match the user's search criteria
            self.findEventsByCriteria(self.eventTCombo_valueSearch.get(),self.ageGroup_valuePRSearch.get())
            #for each event found, append to eventsFoundByCriteria
            for i in self.eventNames:
                self.eventsFoundByCriteria.append(i)
            #call the event combobox method to update
            self.eventCombo()
            #if no events are found, display an error message
            if len(self.eventsFoundByCriteria) < 1:
                errormessage = messagebox.showwarning(title='Error',message='No events found.')
                

    '''This method ensures the user has selected an age group and gender to search for champions. Once the user
       has selected both, the champions will be calculated based on this search'''
    def populateChampionsListbox(self):
        #display an error message if the user hasn't selected an age group or a gender
        if self.ageGroup_valueCSearch .get() == '' or self.gender_valueCSearch.get() == '':
            errormessage = messagebox.showwarning(title='Error',
                                                  message='Please ensure you have used the search boxes above to search for champions.')
        else:
            #calculate the champions based on the selected age group and gender
            self.calcChampions(self.ageGroup_valueCSearch.get(), self.gender_valueCSearch.get())

                
    '''This method calculates the pupils with the most points (champions). A pupil is eligible to be a champion
       if they have participated in 2 track and 1 field, or 2 field and 1 track events, otherwise they will be
       disqualified and not counted. The champions are calculated based on the age group and gender search criteria'''
    def calcChampions(self,ageGroupVal,genderVal):
        #select all the track and field events and pupils by age group and gender
        self.readInChampions(ageGroupVal,genderVal)

        #contains: pupilID,total,Track,Field
        self.scoreArray = []
        for i in self.pupilChamp:
            #append all pupils in database to scoreArray
            self.scoreArray.append([i[0],0,0,0])

        #for each event from the search, find the results for that event
        for event in self.eventsChamp:
            #generate scoreResults and pointResults for that event
            self.incVal = 0

            print('event[4]: ',event[4])
            

            
            self.resultsByEvent(event[1] + ' ' + event[2] + ' ' + event[3])
            print('ALL RESULTS: ',self.results,'FOR:',event,'\n')
            self.sortScorersChamp(event)
            print('SCORERESULTS: ',self.scoreResults,'for :',event)
            print('POINTRESULTS: ',self.pointResults,'for :',event)
            print('\n')
            
            for score in self.scoreResults:
                for pupID in self.scoreArray:
                    if score[0] == pupID[0]:
                        pupID[1] += self.pointResults[self.incVal]
                        self.incVal += 1
                        if event[4] == 'Track':
                            pupID[2] += 1
                        else:
                            pupID[3] += 1

                        for pupil in self.scoreArray:
                            if pupil[2] >= 3 or pupil[3] >= 3:
                                pupil[1] = -1
                            if pupil[2] + pupil[3] > 3:
                                pupil[1] = -1
                        
        #sort the list of champions in descending order
        self.sortChampions(self.scoreArray)
        #used for displaying in listbox
        self.allChampions = []
        #clears champions listboxes
        self.championsNameListbox.delete(0,END)
        self.championsHouseListbox.delete(0,END)
        self.championsPointsListbox.delete(0,END)

        print('\n','SCOREARRY: ',self.scoreArray)




        #if there is no champions, pass
        if len(self.scoreArray) < 0 or self.numChampBox.get() == 0:
            pass
        else:
            #display pupils by the selected number from the spinbox
            for i in range(0,int(self.numChampBox.get())):
                self.allChampions.append(self.scoreArray[i])

        #for each champion
        for i in self.allChampions:
            #if their total is 0, ignore
            if i[1] == 0:
                pass
            else:
                #insert the pupils total in the totals listbox
                self.championsPointsListbox.insert(END,i[1])
                self.cur.execute("SELECT forename,surname,houseName FROM Pupil WHERE pupilID=?", (i[0],))
                self.allChamps = self.cur.fetchall()
                self.champPos = (self.allChampions.index(i)) + 1
                self.championsNameListbox.insert(END, str(self.champPos) + '.' + ' ' + self.allChamps[0][0] + ' ' + self.allChamps[0][1])
                self.championsHouseListbox.insert(END,self.allChamps[0][2])


    '''This method is used to generate 2 lists - scoreResults and pointResults. The list scoreResults will contain
       the elgible pupils who scored for that event (finished in the top 2 scoring results for their house and finished
       within the top 8 scorers for the event. The points can then be assigned to the scorers in assignPoints'''
    def sortScorersChamp(self,event):
        self.scoreResults = []
        self.pointResults = []
        self.guestArray = []
        self.sortList(self.guestArray,self.orderArray[1])
        #sort the list by ascending if the event type is track or relay
        if event[4] == 'Track' or event[4] == 'Relay':
            self.sortList(self.results,self.orderArray[0])
        else:
            #sort the list by descending if not track or relay
            self.sortList(self.results,self.orderArray[1])

        #for each result, if that result matches to a house, append to houseresult
        #for each value in houseresult, if the number of houses is less than 2 append to scoreResults
        #otherwise pupil is a 'guest'                  
        for house in self.houses:
            self.houseresult = []
            for result in self.results:
                if result[3][0] == house:
                    self.houseresult.append(result)
            self.addCount = 0   
            for r in self.houseresult:
                if self.addCount < 2:
                    self.scoreResults.append(r)
                    self.addCount +=1
                else:
                    self.guestArray.append(r)

        #sort the list by ascending if the event type is track or relay
        if event[4] == 'Track' or event[4] == 'Relay':
            self.sortList(self.scoreResults,self.orderArray[0])
        else:
            #sort the list by descending if the event type isn't track or relay
            self.sortList(self.scoreResults,self.orderArray[1])

        #if there is at least 1 scorer for an event, assign the points to the scorers
        if len(self.scoreResults)>0:
            self.assignPoints(self.pointResults,self.scoreResults)
        #number of scorers
        self.numPupils = len(self.pointResults)




    '''This method sorts the list of champions in descending order using a simple bubblesort algorithm'''
    def sortChampions(self,listy):
        sorted = False
        while not sorted:
            sorted = True
            for champ in range(0,len(listy)-1):
                if listy[champ][1] < listy[champ+1][1]:
                    temp = listy[champ]
                    listy[champ] = listy[champ+1]
                    listy[champ+1] = temp
                    sorted = False




                    
class Pupils(Toplevel,mainWindow):
    def __init__(self):
        #Open database
        database_opts.__init__(self)
        '''create string variables, gender array and order arrays, also read in values'''
        self.forenameVal = StringVar()
        self.surnameVal = StringVar()
        self.genderVal = StringVar()
        self.genderValues = ['Male','Female']
        self.orderValues = ['Forename','Surname']
        self.pupilSearchValuesPupils = []
        self.forenameValEdit = StringVar()
        self.surnameValEdit = StringVar()
        self.genderValEdit = StringVar()
        self.pupilLabelVal = StringVar()
        self.pupilsFound = StringVar()
        #Read in values from database
        self.readInHouses()
        self.readInPupils()
        self.readInEvents()
        self.readInAgeGroups()

    def configPupilsScreen(self,master):
        #Make current window 'TopLevel'
        Toplevel.__init__(self)

        #Define window
        self.master = master
        self.title('Pupil Screen')
        self.geometry('590x480+300+20')
        self.pupilLabelVal.set('No Pupil Selected')
        self.resizable(0,0)
        #Notebook widget on window
        self.master.configure(background=self.rgb_light_green)
        self.note = ttk.Notebook(self, width=590,height=480)
        self.viewPupilTab = Frame(self.note,bg=self.rgb_light_green)
        self.addPupilTab = Frame(self.note,bg=self.rgb_light_green)
        self.editPupilTab = Frame(self.note,bg=self.rgb_light_green)
        #Add tabs to notebook widget
        self.note.add(self.viewPupilTab, text='View Pupils')
        self.note.add(self.addPupilTab, text='Add Pupils')
        self.note.add(self.editPupilTab, text='Edit Pupil')
        self.note.pack()


        #View Pupils Window
        #Frames
        self.vpfPupilTop = Frame(self.viewPupilTab,bg=self.rgb_light_green,pady=10)
        self.vpfPupilTop.pack(side=TOP)
        self.vpfPupilLeft = Frame(self.viewPupilTab,bg=self.rgb_light_green,padx=20)
        self.vpfPupilLeft.pack(side=LEFT)

        #Top frame - title for view pupils
        self.viewPupilsLabel = Label(self.vpfPupilTop, text='View Pupils',relief=RAISED,bg=self.rgb_light_green,font=('Courier',24)).grid(row=0,column=0)

        #Left frame - search for pupils using comboboxes and display in listbox
        self.genderSearchLabel = Label(self.vpfPupilLeft, text='Gender: ',bg=self.rgb_light_green).grid(row=0,column=0)
        self.ageSearchLabel = Label(self.vpfPupilLeft, text='Age Group: ',bg=self.rgb_light_green).grid(row=0,column=1)
        self.houseSearchLabel = Label(self.vpfPupilLeft, text='House: ',bg=self.rgb_light_green).grid(row=0,column=2)
        self.orderByLabel = Label(self.vpfPupilLeft,text='Order By:',bg=self.rgb_light_green).grid(row=0,column=3,pady=10)
        self.applySearchButton = Button(self.vpfPupilLeft, text='Search', command=self.applySearchPupil,
                                        highlightbackground=self.rgb_light_green).grid(row=2,column=2,pady=8)
        self.applyOrderButton = Button(self.vpfPupilLeft, text='Order', command=self.applyOrderPupil,
                                       highlightbackground=self.rgb_light_green).grid(row=2,column=3,pady=8)
        self.pupilsFound.set('No Pupils Found')
        self.pupilNamesLabel = Label(self.vpfPupilLeft, textvariable=self.pupilsFound,bg=self.rgb_light_green).grid(row=3,column=1,columnspan=2)
        self.pupilNamesListbox = Listbox(self.vpfPupilLeft,width=20,height=13)
        self.pupilNamesListbox.grid(row=4,column=1,rowspan=4,columnspan=2,pady=(0,8),padx=10)
        self.editPupilButton = Button(self.vpfPupilLeft, text='Edit Selection', command=self.editPupil,
                                      highlightbackground=self.rgb_light_green).grid(row=5,column=3,pady=(0,10))
        self.deletePupilButton = Button(self.vpfPupilLeft, text='Delete Selection', command=self.deletePupil,
                                        highlightbackground=self.rgb_light_green).grid(row=6,column=3,pady=(0,10))


        #Add Pupil Window
        #Frames
        self.apfPupilTop = Frame(self.addPupilTab,bg=self.rgb_light_green,pady=10)
        self.apfPupilTop.pack(side=TOP)
        self.apfPupilLeft = Frame(self.addPupilTab,bg=self.rgb_light_green)
        self.apfPupilLeft.pack(side=LEFT,padx=120)

        #Top frame - title for add pupils
        self.addPupilLabel= Label(self.apfPupilTop, text='Add Pupil',relief=RAISED,bg=self.rgb_light_green,font=('Courier',24)).grid(row=0,column=0)

        #Left frame - add pupil widgets on add pupil tab
        self.foreLabel = Label(self.apfPupilLeft, text='Forename:',bg=self.rgb_light_green).grid(row=0,column=0)
        self.surLabel = Label(self.apfPupilLeft, text='Surname:',bg=self.rgb_light_green).grid(row=1,column=0)
        self.genderLabel = Label(self.apfPupilLeft, text='Gender:',bg=self.rgb_light_green).grid(row=2,column=0)
        self.ageGroupLabel = Label(self.apfPupilLeft, text='Age Group:',bg=self.rgb_light_green).grid(row=4,column=0)
        self.houseLabel = Label(self.apfPupilLeft, text='House:',bg=self.rgb_light_green).grid(row=5,column=0)
        self.entryFore = Entry(self.apfPupilLeft, textvariable=self.forenameVal).grid(row=0,column=1)
        self.entrySur = Entry(self.apfPupilLeft, textvariable=self.surnameVal).grid(row=1,column=1)
        self.radGenderMale = Radiobutton(self.apfPupilLeft, value='Male', variable=self.genderVal, text='Male',
                                         bg=self.rgb_light_green).grid(row=2,column=1)
        self.radGenderFemale = Radiobutton(self.apfPupilLeft, value='Female', variable=self.genderVal, text='Female',
                                           bg=self.rgb_light_green).grid(row=3,column=1)
        self.genderVal.set('Male')
        self.applyButton = Button(self.apfPupilLeft, command=self.applyAddPupil, text='Apply',
                                  highlightbackground=self.rgb_light_green).grid(row=6,column=1,pady=(20,0))
        self.importPupilsButton = Button(self.apfPupilLeft, text='Import File',command=self.importPupils,
                                         highlightbackground=self.rgb_light_green).grid(row=7,column=1,pady=(5,0))


        #Edit Pupil Window
        #Frames
        self.epfPupilTop = Frame(self.editPupilTab,bg=self.rgb_light_green)
        self.epfPupilTop.pack(side=TOP,pady=(10,0))
        self.epfPupilLeft = Frame(self.editPupilTab,bg=self.rgb_light_green)
        self.epfPupilLeft.pack(side=LEFT,padx=100)

        #Top frame - display title
        self.editPupilLabel= Label(self.epfPupilTop, text='Edit Pupil',relief=RAISED,bg=self.rgb_light_green,font=('Courier',24)).grid(row=0,column=0)

        #Left frame - edit pupil widgets on edit pupil tab
        self.pupilEditLabel = Label(self.epfPupilLeft, text='Currently Editing: ',bg=self.rgb_light_green).grid(row=0,column=0,pady=20)
        self.pupilNameLabel = Label(self.epfPupilLeft, textvariable=self.pupilLabelVal,bg=self.rgb_light_green).grid(row=0,column=1)
        self.foreEditLabel = Label(self.epfPupilLeft, text='Forename:',bg=self.rgb_light_green).grid(row=1,column=0)   
        self.surEditLabel = Label(self.epfPupilLeft, text='Surname:',bg=self.rgb_light_green).grid(row=2,column=0)
        self.genderEditLabel = Label(self.epfPupilLeft, text='Gender:',bg=self.rgb_light_green).grid(row=3,column=0)
        self.ageGroupEditLabel = Label(self.epfPupilLeft, text='Age Group:',bg=self.rgb_light_green).grid(row=5,column=0)
        self.houseEditLabel = Label(self.epfPupilLeft, text='House:',bg=self.rgb_light_green).grid(row=6,column=0)
        self.entryFore = Entry(self.epfPupilLeft, textvariable=self.forenameValEdit).grid(row=1,column=1)
        self.entrySur = Entry(self.epfPupilLeft, textvariable=self.surnameValEdit).grid(row=2,column=1)
        self.radGenderMale = Radiobutton(self.epfPupilLeft, value='Male', variable=self.genderValEdit, text='Male',
                                         bg=self.rgb_light_green).grid(row=3,column=1)
        self.radGenderFemale = Radiobutton(self.epfPupilLeft, value='Female', variable=self.genderValEdit, text='Female',
                                           bg=self.rgb_light_green).grid(row=4,column=1)
        self.genderValEdit.set('Male')
        self.applyChangesButton = Button(self.epfPupilLeft, command=self.applyEditPupil, text='Apply Changes',
                                         highlightbackground=self.rgb_light_green).grid(row=7,column=1,pady=(10,0))

        #combobox procedures and reset the pupil listbox
        self.genderCombo()
        self.ageGroupCombo()
        self.houseCombo()
        self.orderByCombo()
        self.resetListboxPupil()


    #Comboboxes
    def genderCombo(self):
        #view pupils gender combobox
        self.gender_valuePS = StringVar()
        self.genderComboboxPS = ttk.Combobox(self.vpfPupilLeft, textvariable=self.gender_valuePS,width=10,state='readonly')
        self.genderComboboxPS['values'] = (self.genderValues)
        self.genderComboboxPS.grid(row=1,column=0,padx=6)
        
    def ageGroupCombo(self):
        #view pupils age group combobox
        self.ageGroup_valuePS = StringVar()
        self.ageGroupComboPS = ttk.Combobox(self.vpfPupilLeft, textvariable=self.ageGroup_valuePS,width=10,state='readonly')
        self.ageGroupComboPS['values'] = (self.ageGroupArray)
        self.ageGroupComboPS.grid(row=1,column=1,padx=10)
        #add pupils age group combobox
        self.ageGroup_valueAP = StringVar()
        self.ageGroupComboAP = ttk.Combobox(self.apfPupilLeft, textvariable=self.ageGroup_valueAP,state='readonly')
        self.ageGroupComboAP['values'] = (self.ageGroupArray)
        self.ageGroupComboAP.grid(row=4,column=1)
        #edit pupils age group combobox
        self.ageGroup_valueEP = StringVar()
        self.ageGroupComboEP = ttk.Combobox(self.epfPupilLeft, textvariable=self.ageGroup_valueEP,state='readonly')
        self.ageGroupComboEP['values'] = (self.ageGroupArray)
        self.ageGroupComboEP.grid(row=5,column=1)

    def houseCombo(self):
        #view pupils house name combobox
        self.houseCombo_valuePS = StringVar()
        self.houseComboboxPS = ttk.Combobox(self.vpfPupilLeft, textvariable=self.houseCombo_valuePS,width=10,state='readonly')
        self.houseComboboxPS['values'] = (self.houseArray)
        self.houseComboboxPS.grid(row=1,column=2,padx=10)
        #add pupils house name combobox
        self.houseCombo_valueAP = StringVar()
        self.houseComboboxAP = ttk.Combobox(self.apfPupilLeft, textvariable=self.houseCombo_valueAP,state='readonly')
        self.houseComboboxAP['values'] = (self.houseArray)
        self.houseComboboxAP.grid(row=5,column=1)
        #edit pupils house name combobox
        self.houseCombo_valueEP = StringVar()
        self.houseComboboxEP = ttk.Combobox(self.epfPupilLeft, textvariable=self.houseCombo_valueEP,state='readonly')
        self.houseComboboxEP['values'] = (self.houseArray)
        self.houseComboboxEP.grid(row=6,column=1)

    def orderByCombo(self):
        #view pupils order combobox
        self.orderCombo_valueVP = StringVar()
        self.orderComboVP = ttk.Combobox(self.vpfPupilLeft, textvariable=self.orderCombo_valueVP,width=10,state='readonly')
        self.orderComboVP['values'] = (self.orderValues)
        self.orderComboVP.grid(row=1,column=3,padx=10)


    '''This method checks for an entered forename and surname for a pupil to add. It checks to see if it is 'correct'
       and text only. Once a user has entered a forename and surname, and selected a gender, house name and age group
       the pupil will be added into the database and the widgets will reset'''
    def applyAddPupil(self):
        #checks forename and surname for numbers
        self.foreAddCheck = self.forenameVal.get().isdigit()
        self.surAddCheck = self.surnameVal.get().isdigit()
        #if forename or surname isn't text, display an error message
        if self.foreAddCheck == True or self.surAddCheck == True:
            errormessage = messagebox.showwarning(title='Error',message='Please ensure your entry is text only.')
        else:
            #if the user hasn't used all of the boxes to input a pupil's data, an error message is displayed
            if self.forenameVal.get() == '' or self.surnameVal.get() == '' or self.ageGroup_valueAP.get() == '' or self.houseCombo_valueAP.get() == '':
                errorMessage = messagebox.showwarning(title='Error',message='Please ensure all fields have been completed.')
            else:
                #range check for forename and surname
                if len(self.forenameVal.get()) + len(self.surnameVal.get()) >= 30:
                    checkMessage = messagebox.showinfo(title='Error',message='You have entered a large name. Please ensure this is correct.')
                #confirmation message for adding a pupil
                myMessage = messagebox.askyesno(title='Add Pupil',message='Details: \n \n Forename:  %s \n Surname:  %s \n Gender:  %s \n Age Group:  %s \n House:  %s \n \n Is this correct?'
                                                %(self.forenameVal.get(), self.surnameVal.get(),self.genderVal.get(),self.ageGroup_valueAP.get(),self.houseComboboxAP.get()))
                if myMessage == 1:
                    #add pupil to database
                    self.insertPupil(self.forenameVal.get(),self.surnameVal.get(),self.genderVal.get(),self.ageGroup_valueAP.get(),self.houseCombo_valueAP.get())
                    #reset forename and surname entry
                    self.forenameVal.set('')
                    self.surnameVal.set('')

                
    '''This method takes the user's gender, age group, house and order selection from the comboboxes.
       If a value isn't present in all the comboboxes an error will be displayed, if not pupils
       are found based on the search criteria and the number of pupils label will update'''
    def applySearchPupil(self):
        #order the pupils
        self.orderVal = self.orderCombo_valueVP.get()
        #contains all pupils once search method is called
        self.pupilSearchValuesPupils = []
        #if user hasn't selected values, display an error message
        if self.gender_valuePS.get() == '' or self.ageGroup_valuePS.get() == '' or self.houseCombo_valuePS.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='Please ensure you have used all the boxes for your search.')
        else:
            #rest the listbox
            self.resetListboxPupil()
            #search for pupil's based on the search criteria
            self.applySearchCriteriaPupil(self.gender_valuePS.get(),self.ageGroup_valuePS.get(),self.houseCombo_valuePS.get())
            #update the pupils found label (number)
            self.pupilsFound.set(str(len(self.pupilSearchValuesPupils)) + " " + "pupil(s) found.")


    '''This method searches for pupils based on the user's search criteria'''
    def applySearchCriteriaPupil(self,genderValSearchP,ageGroupSearchP,houseValSearchP):
        #searches for pupils by gender, age group and house name
        self.searchForPupilP(self.gender_valuePS.get(),self.ageGroup_valuePS.get(),self.houseCombo_valuePS.get())
        #resets the pupil listbox
        self.resetListboxPupil()
        #for each pupil found, insert into pupil listbox
        for i in self.pupilSearchValuesPupils:
            self.pupilNamesListbox.insert(END, i[1] + ' ' + i[2])


    '''This method finds the pupils by search criteria then calls the order method'''
    def applyOrderPupil(self):
        #gets the order selection
        self.orderVal = self.orderCombo_valueVP.get()
        #display an error message if no order value has been selected
        if self.orderVal == '':
            errormessage = messagebox.showwarning(title='Error',message='Please ensure you have selected to order by forename or surname.')
        else:
            #reset the pupil listbox
            self.resetListboxPupil()
            #find pupils by the search criteria
            self.applySearchCriteriaPupil(self.gender_valuePS.get(),self.ageGroup_valuePS.get(),self.houseCombo_valuePS.get())
            #update the pupils found label (number)
            self.pupilsFound.set(str(len(self.pupilSearchValuesPupils)) + " " + "pupil(s) found.")
            #order the found pupils by the order selection
            self.orderSearchPupil(self.gender_valuePS,self.ageGroup_valuePS,self.houseCombo_valuePS)


    '''This method orders the found pupils by the order selection'''
    def orderSearchPupil(self,genderValSearchP,ageGroupSearchP,houseValSearchP):
        #gets the order selection
        self.orderVal = self.orderCombo_valueVP.get()
        #if the selection is forename, order the found pupils by forename
        if self.orderVal == 'Forename':
            self.orderFore(self.gender_valuePS.get(),self.ageGroup_valuePS.get(),self.houseCombo_valuePS.get())
        #if the selection is surname, order the found pupils by surname
        elif self.orderVal == 'Surname':
            self.orderSur(self.gender_valuePS.get(),self.ageGroup_valuePS.get(),self.houseCombo_valuePS.get())
        #reset the listbox
        self.resetListboxPupil()
        #find all the pupils
        self.pupilSearchValuesPupils = self.cur.fetchall()
        #add each pupil to the listbox
        for i in self.pupilSearchValuesPupils:
            self.pupilNamesListbox.insert(END, i[1] + ' ' + i[2])
        #update the pupils found label (number)
        self.pupilsFound.set(str(len(self.pupilSearchValuesPupils)) + " " + "pupil(s) found.")

        
    '''This method resets the pupils listbox'''
    def resetListboxPupil(self):
        #clear the pupil listbox
        self.pupilNamesListbox.delete(0,END)


    '''This method gets the selected pupil to delete from the pupils listbox and calls the method to delete the
       pupil from the database. The listbox will update to match the previous search criteria'''
    def deletePupil(self):
        #display an error if a pupil hasn't been selected
        if self.pupilNamesListbox.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='Please ensure you have selected a pupil to delete.')
        else:
            #get the selected pupil
            self.selectedPupil = self.pupilNamesListbox.curselection()[0]
            self.selectedPupilVal = (self.pupilSearchValuesPupils[self.selectedPupil])
            #get the pupils ID
            self.pupil_idDel = self.selectedPupilVal[0]
            #confirmation message
            myMessage = messagebox.askyesno(title='Delete Pupil',message='Are you sure you wish to delete your selection?')
            if myMessage == 1:
                #delete the pupil from the database from the pupilID
                self.removePupil(self.pupil_idDel)
                #delete pupil from the listbox
                self.pupilNamesListbox.delete(END,self.selectedPupil)
                #re-search by previous search criteria
                self.applySearchPupil()
                self.applySearchCriteriaPupil(self.gender_valuePS,self.ageGroup_valuePS,self.houseCombo_valuePS)


    '''This method gets the selected pupil to edit from the listbox and updates the edit pupils tab to match
       the details of the selected pupil'''
    def editPupil(self):
        #display an error message if no pupil has been selected
        if self.pupilNamesListbox.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='Please ensure you have selected a pupil to edit.')
        else:
            #inform user of updated edit pupils tab
            myMessage = messagebox.showinfo(title='Pupil Selected',message='Go to the Edit Pupil tab to edit: \n%s.'
                                            %(self.pupilNamesListbox.get(ACTIVE),))
            #get the selected pupil
            self.selectedPupilToEdit = (self.pupilNamesListbox.curselection()[0])
            #find the pupils details
            self.selectedPupilValToEdit = (self.pupilSearchValuesPupils[self.selectedPupilToEdit])
            #set the forename and surname entry boxes to the pupils forename and surname
            self.forenameValEdit.set(self.selectedPupilValToEdit[1])
            self.surnameValEdit.set(self.selectedPupilValToEdit[2])
            #update the pupil editing label
            self.pupilLabelVal.set(self.pupilNamesListbox.get(ACTIVE))
        
       
    '''This method gets the user's entry for the pupil to edit and checks for any entry that isn't text, then
       updates the pupils details in the database and the pupils listbox on the view pupils tab is updated'''
    def applyEditPupil(self):
        #if no pupil has been selected, display an error message
        if self.pupilLabelVal.get() == 'No Pupil Selected':
            errormessage = messagebox.showwarning(title='Error',message='Please ensure you have selected a pupil to edit.')
            #reset the entry boxes and comboboxes
            self.forenameValEdit.set('')
            self.surnameValEdit.set('')
            self.ageGroup_valueEP.set('')
            self.houseCombo_valueEP.set('')
        else:
            #check for entry that isn't text
            self.foreEditCheck = self.forenameValEdit.get().isdigit()
            self.surEditCheck = self.surnameValEdit.get().isdigit()
            #display an error if the entry isn't text
            if self.foreEditCheck == True or self.surEditCheck == True:
                errormessage = messagebox.showwarning(title='Error',message='Please ensure your entry is text only.')
            else:
                #checks for values entered in the entry boxes and comboboxes
                if self.forenameValEdit.get() == '' or self.surnameValEdit.get() == '' or self.genderValEdit.get() == '' or self.ageGroup_valueEP.get() == '' or self.houseComboboxEP.get() == '':
                    errormessage = messagebox.showwarning(title='Error',message='Please ensure you have used all of the boxes to edit a pupil.')
                else:
                    #range check for forename and surname entry
                    if len(self.forenameValEdit.get()) + len(self.surnameValEdit.get()) >= 30:
                        checkMessage = messagebox.showinfo(title='Error',message='You have entered a large name. Please ensure this is correct.')
                    #confirmation message
                    myMessage = messagebox.askyesno(title='Edit Pupil',message='New Pupil details: \n \n Forename:  %s \n Surname:  %s \n Gender:  %s \n Age Group:  %s \n House:  %s \n \n Is this correct?'
                                                    %(self.forenameValEdit.get(),self.surnameValEdit.get(),self.genderValEdit.get(),self.ageGroup_valueEP.get(),self.houseCombo_valueEP.get(),))
                    if myMessage == 1:
                        #get the selected pupils ID
                        self.existingID = self.selectedPupilValToEdit[0]
                        #call the update method
                        self.updatePupilDetails(self.forenameValEdit.get(),self.surnameValEdit.get(),self.genderValEdit.get(),self.ageGroup_valueEP.get(),self.houseCombo_valueEP.get(),self.existingID)
                        #reset the listbox
                        self.resetListboxPupil()
                        #reset window
                        self.pupilLabelVal.set('No Pupil Selected')
                        self.forenameValEdit.set('')
                        self.surnameValEdit.set('')
                        self.ageGroup_valueEP.set('')
                        self.houseCombo_valueEP.set('')
                        #re-search from previous search criteria
                        self.applySearchPupil()




                        
class Events(Toplevel,mainWindow):
    def __init__(self):
        #Open database
        database_opts.__init__(self)
        #create string variables for entry, arrays and read in values
        self.eventName = StringVar()
        self.eventNameVal = StringVar()
        self.eventTypes = ['Relay','Track','Field']
        self.resultTypes = ['metres','seconds','minutes']
        self.genderValuesEvents = ['Male','Female']
        self.eventsFoundVar = StringVar()
        #Read in values from database
        self.readInHouses()
        self.readInPupils()
        self.readInEvents()
        self.readInAgeGroups()
        self.readInCupComps()
        
              
    def configEventsScreen(self,master):
        #Make current window 'TopLevel'
        Toplevel.__init__(self)

        #Define window
        self.master = master
        self.title('Event Screen')
        self.geometry('600x510+300+20')
        self.resizable(0,0)
        self.master.configure(background=self.rgb_light_green)

        #Notebook widget on window
        self.note = ttk.Notebook(self, width=600,height=510)
        self.viewEventsTab = Frame(self.note,bg=self.rgb_light_green)
        self.addEventsTab = Frame(self.note,bg=self.rgb_light_green)
        #Add tabs to notebook widget
        self.note.add(self.viewEventsTab, text='View Events')
        self.note.add(self.addEventsTab, text='Add Event')
        self.note.pack()
    

        #View Events Window
        #Frames
        self.vefEventTop = Frame(self.viewEventsTab,bg=self.rgb_light_green,pady=10)
        self.vefEventTop.pack(side=TOP)
        self.vefEventLeft = Frame(self.viewEventsTab,bg=self.rgb_light_green,padx=20)
        self.vefEventLeft.pack(side=LEFT)
        
        #Top frame - title for view events
        self.currentEventsLabel = Label(self.vefEventTop, text='View Events',bg=self.rgb_light_green,
                                        relief=RAISED,font=('Courier',24)).grid(row=0,column=0)        
        self.sortByLabel = Label(self.vefEventTop, text='Sort By:',
                                 bg=self.rgb_light_green).grid(row=1,column=0,pady=(10,0))

        #Left frame - search for events
        self.ageGroupSLabel = Label(self.vefEventLeft, text='Age Group:',
                                    bg=self.rgb_light_green).grid(row=0,column=0)
        self.genderSLabel = Label(self.vefEventLeft, text='Gender:',
                                  bg=self.rgb_light_green).grid(row=0,column=1)
        self.eventTypeSLabel = Label(self.vefEventLeft, text='Event Type:',
                                     bg=self.rgb_light_green).grid(row=0,column=2)
        self.cupCompSLabel = Label(self.vefEventLeft, text='Cup Competition:',
                                   bg=self.rgb_light_green).grid(row=0,column=3)
        self.applyAgeGBut = Button(self.vefEventLeft,text='Apply',command=self.searchByAG,
                                   highlightbackground=self.rgb_light_green).grid(row=2,column=0)
        self.applyGenderBut = Button(self.vefEventLeft,text='Apply',command=self.searchByG,
                                     highlightbackground=self.rgb_light_green).grid(row=2,column=1)
        self.applyETypeBut = Button(self.vefEventLeft,text='Apply',command=self.searchByEType,
                                    highlightbackground=self.rgb_light_green).grid(row=2,column=2)
        self.applyCCompBut = Button(self.vefEventLeft,text='Apply',command=self.searchByCComp,
                                    highlightbackground=self.rgb_light_green).grid(row=2,column=3)
        self.applyAllBut = Button(self.vefEventLeft,text='Apply All',command=self.searchByAll,
                                  highlightbackground=self.rgb_light_green).grid(row=3,column=3,pady=(10,0))
        self.eventsFoundVar.set('No Events Found')
        self.eventsFoundLabel = Label(self.vefEventLeft,textvariable=self.eventsFoundVar,
                                      bg=self.rgb_light_green).grid(row=3,column=1,pady=(15,0))
        self.currentEventsListbox = Listbox(self.vefEventLeft, width=30, height=12)
        self.currentEventsListbox.grid(row=4,column=0,pady=5,columnspan=3,padx=20)
        self.deleteEventBut = Button(self.vefEventLeft, text='Delete Selection', command=self.deleteEvent,
                                     highlightbackground=self.rgb_light_green).grid(row=4,column=3,columnspan=2)
        

        #Add Events Window
        #Frames
        self.aefEventTop = Frame(self.addEventsTab,bg=self.rgb_light_green)
        self.aefEventTop.pack(side=TOP)
        self.aefEventMiddle = Frame(self.addEventsTab,bg=self.rgb_light_green,pady=40)
        self.aefEventMiddle.pack()

        #Top frame - title for add events
        self.addEventsLabel = Label(self.aefEventTop, text='Add Events',bg=self.rgb_light_green,
                                    relief=RAISED,font=('Courier',24)).grid(row=0,column=0,pady=10)

        #Middle frame - add an event
        self.eventNameLabel = Label(self.aefEventMiddle, text='Event Name:',
                                    bg=self.rgb_light_green).grid(row=0,column=0,pady=40,padx=10)
        self.ageGroupLabel = Label(self.aefEventMiddle, text='Age Group:',
                                   bg=self.rgb_light_green).grid(row=1,column=0,padx=10)
        self.genderLabel = Label(self.aefEventMiddle, text='Gender:',
                                 bg=self.rgb_light_green).grid(row=2,column=0,padx=10)
        self.genderLabel = Label(self.aefEventMiddle, text='Event Type:',
                                 bg=self.rgb_light_green).grid(row=3,column=0,padx=10)
        self.resultTypeLabel = Label(self.aefEventMiddle, text='Result Type:',
                                     bg=self.rgb_light_green).grid(row=4,column=0,padx=10)
        self.cupCompLabel = Label(self.aefEventMiddle, text='Cup Competition:',
                                  bg=self.rgb_light_green).grid(row=5,column=0,padx=10,pady=10)
        self.eventNameEntry = Entry(self.aefEventMiddle, textvariable=self.eventName).grid(row=0,column=1)
        self.applyBut = Button(self.aefEventMiddle, text='ADD', command=self.applyAddEvent,
                               highlightbackground=self.rgb_light_green).grid(row=6,column=1,pady=10)
        self.clearBut = Button(self.aefEventMiddle,text='Clear',command=self.clearEntry,
                               highlightbackground=self.rgb_light_green).grid(row=7,column=1)

        #combobox procedures
        self.ageGroupCombo()
        self.genderCombo()
        self.eventTypeCombo()
        self.resultTypeCombo()
        self.cupCompCombo()


    #Comboboxes
    def ageGroupCombo(self):
        #view events age group combobox
        self.ageGroup_valueVE = StringVar()
        self.ageGroupComboboxVE = ttk.Combobox(self.vefEventLeft,
                                               textvariable=self.ageGroup_valueVE,width=10,state='readonly')
        self.ageGroupComboboxVE['values'] = (self.ageGroupArray)
        self.ageGroupComboboxVE.grid(row=1,column=0,padx=10)
        #add events age group combobox
        self.ageGroup_valueSE = StringVar()
        self.ageGroupComboboxSE = ttk.Combobox(self.aefEventMiddle,
                                               textvariable=self.ageGroup_valueSE,state='readonly')
        self.ageGroupComboboxSE['values'] = (self.ageGroupArray)
        self.ageGroupComboboxSE.grid(row=1,column=1)
        
    def genderCombo(self):
        #view events gender combobox
        self.gender_valueVE = StringVar()
        self.genderComboboxVE = ttk.Combobox(self.vefEventLeft,
                                             textvariable=self.gender_valueVE,width=10,state='readonly')
        self.genderComboboxVE['values'] = (self.genderValuesEvents)
        self.genderComboboxVE.grid(row=1,column=1,padx=10)
        #add events gender combobox
        self.gender_value = StringVar()
        self.genderCombobox = ttk.Combobox(self.aefEventMiddle,
                                           textvariable=self.gender_value,state='readonly')
        self.genderCombobox['values'] = (self.genderValuesEvents)
        self.genderCombobox.grid(row=2,column=1)
         
    def eventTypeCombo(self):
        #view events event type combobox
        self.eventType_valueVE = StringVar()
        self.eventTypeComboboxVE = ttk.Combobox(self.vefEventLeft,
                                                textvariable=self.eventType_valueVE,width=10,state='readonly')
        self.eventTypeComboboxVE['values'] = (self.eventTypes)
        self.eventTypeComboboxVE.grid(row=1,column=2,padx=10)
        #add events event type combobox
        self.eventType_value = StringVar()
        self.eventTypeCombobox = ttk.Combobox(self.aefEventMiddle,
                                              textvariable=self.eventType_value,state='readonly')
        self.eventTypeCombobox['values'] = (self.eventTypes)
        self.eventTypeCombobox.grid(row=3,column=1)
        
    def resultTypeCombo(self):
        #add events result type combobox
        self.resultType_value = StringVar()
        self.resultTypeCombobox = ttk.Combobox(self.aefEventMiddle,
                                               textvariable=self.resultType_value,state='readonly')
        self.resultTypeCombobox['values'] = (self.resultTypes)
        self.resultTypeCombobox.grid(row=4,column=1)

    def cupCompCombo(self):
        #view events cup comptition combobox
        self.cupCombo_valueVE = StringVar()
        self.cupCompComboboxVE = ttk.Combobox(self.vefEventLeft,
                                              textvariable=self.cupCombo_valueVE,width=10,state='readonly')
        self.cupCompComboboxVE['values'] = (self.cupCompArray)
        self.cupCompComboboxVE.grid(row=1,column=3)
        #add events cup comptition combobox
        self.cupCombo_value = StringVar()
        self.cupCompCombobox = ttk.Combobox(self.aefEventMiddle,
                                            textvariable=self.cupCombo_value,state='readonly')
        self.cupCompCombobox['values'] = (self.cupCompArray)
        self.cupCompCombobox.grid(row=5,column=1)


    '''This method clears all the entry box variables'''
    def clearEntry(self):
        #clear the users entry boxes
        self.eventName.set('')
        self.ageGroup_valueSE.set('')
        self.gender_value.set('')
        self.eventType_value.set('')
        self.resultType_value.set('')
        self.cupCombo_value.set('')


    '''This method allows the user to search for events by age group'''
    def searchByAG(self):
        #display an error if no age group is selected
        if self.ageGroup_valueVE.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='No age group selected.')
        else:
            #clear the listbox
            self.currentEventsListbox.delete(0,END)
            #find events by age group search criteria
            self.findEventsByAG(self.ageGroup_valueVE.get())
            #insert all events found into listbox
            for i in self.eventsFound:
                self.currentEventsListbox.insert(END,i)
            #update events found label (number)
            self.eventsFoundVar.set(str(len(self.eventsFound)) + " " + "event(s) found.")
            #reset search boxes
            self.gender_valueVE.set('')
            self.eventType_valueVE.set('')
            self.cupCombo_valueVE.set('')


    '''This method allows the user to search for events by gender'''
    def searchByG(self):
        #display an error if no gender is selected
        if self.gender_valueVE.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='No gender selected.')
        else:
            #clear the listbox
            self.currentEventsListbox.delete(0,END)
            #find events by gender search criteria
            self.findEventsByG(self.gender_valueVE.get())
            #insert all events found into listbox
            for i in self.eventsFound:
                self.currentEventsListbox.insert(END,i)
            #update events found label (number)
            self.eventsFoundVar.set(str(len(self.eventsFound)) + " " + "event(s) found.")
            #reset search boxes
            self.ageGroup_valueVE.set('')
            self.eventType_valueVE.set('')
            self.cupCombo_valueVE.set('')


    '''This method allows the user to search for events by event type'''
    def searchByEType(self):
        #display an error if no event type is selected
        if self.eventType_valueVE.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='No event type selected.')
        else:
            #clear the listbox
            self.currentEventsListbox.delete(0,END)
            #find events by event type search criteria
            self.findEventsByEType(self.eventType_valueVE.get())
            #insert all events found into listbox
            for i in self.eventsFound:
                self.currentEventsListbox.insert(END,i)
            #update events found label (number)
            self.eventsFoundVar.set(str(len(self.eventsFound)) + " " + "event(s) found.")
            #reset search boxes
            self.ageGroup_valueVE.set('')
            self.gender_valueVE.set('')
            self.cupCombo_valueVE.set('')
     

    '''This method allows the user to search for events by cup competition'''           
    def searchByCComp(self):
        #display an error if no cup competition is selected
        if self.cupCombo_valueVE.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='No cup competition selected.')
        else:
            #clear the listbox
            self.currentEventsListbox.delete(0,END)
            #find events by cup competition search criteria
            self.findEventsByCComp(self.cupCombo_valueVE.get())
            #insert all events found into listbox
            for i in self.eventsFound:
                self.currentEventsListbox.insert(END,i)
            #update events found label (number)
            self.eventsFoundVar.set(str(len(self.eventsFound)) + " " + "event(s) found.")
            #reset search boxes
            self.ageGroup_valueVE.set('')
            self.gender_valueVE.set('')
            self.eventType_valueVE.set('')


    '''This method allows the user to search for events by all criteria'''           
    def searchByAll(self):
        #display an error if all search boxes don't have a value
        if self.ageGroup_valueVE.get() == '' or self.gender_valueVE.get() == '' or self.eventType_valueVE.get() == '' or self.cupCombo_valueVE.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='Please ensure all fields have been completed.')
        else:
            #clear the listbox
            self.currentEventsListbox.delete(0,END)
            #find events by all search criteria
            self.findEventsByAll(self.ageGroup_valueVE.get(),self.gender_valueVE.get(),self.eventType_valueVE.get(),self.cupCombo_valueVE.get())
            #insert all events found into listbox
            for i in self.eventsFound:
                self.currentEventsListbox.insert(END,i)
            #update events found label (number)
            self.eventsFoundVar.set(str(len(self.eventsFound)) + " " + "event(s) found.")

        
    '''This method allows the user to add an event into the database. The user must have entered values into
       every entry box otherwise an error is displayed'''
    def applyAddEvent(self):
        #display an error if user hasn't entered a value in every entry box
        if self.eventName.get() == '' or self.ageGroup_valueSE.get() == '' or self.gender_value.get() == '' or self.eventType_value.get() == '' or self.resultType_value == '' or self.cupCombo_value.get() == '': 
            errormessage = messagebox.showwarning(title='Error',message='Please ensure all fields have been completed.')
        else:
            self.existVal = 0
            #check for an existing event in the list of all events from the input
            for i in self.eventArraySeparate:
                if i[1] == self.eventName.get() and i[2] == self.ageGroup_valueSE.get() and i[3] == self.gender_value.get() and i[4] == self.eventType_value.get() and i[5] == self.resultType_value.get() and i[6] == self.cupCombo_value.get():
                    self.existVal += 1
            #display an error if the event already exists
            if self.existVal > 0:
                errormessage = messagebox.showwarning(title='Error',message='Event already exists.')
            else:
                #range check for event name entry
                if len(self.eventName.get()) > 15:
                    checkMessage = messagebox.showinfo(title='Error',message='You have entered a large name. Please ensure this is correct.')
                #confirmation message
                myMessage = messagebox.askyesno(title='Add Event',message='Details: \n \n Name: %s \n Age Group: %s \n Gender: %s \n Event Type: %s \n Result Type: %s \n Cup Comp: %s \n \n Is this correct?'
                                                %(self.eventName.get(),self.ageGroup_valueSE.get(),self.gender_value.get(),self.eventType_value.get(),self.resultType_value.get(),self.cupCombo_value.get(),))
                if myMessage == 1:
                    #add event to database
                    self.insertEvent(self.eventName.get(),self.ageGroup_valueSE.get(),self.gender_value.get(),self.eventType_value.get(),self.resultType_value.get(),self.cupCombo_value.get())
                    #clear contents of listbox
                    self.currentEventsListbox.delete(0,END)
                    #reset widgets on add events window
                    self.eventsFoundVar.set('No Events Found')
                    self.ageGroup_valueVE.set('')
                    self.gender_valueVE.set('')
                    self.eventType_valueVE.set('')
                    self.cupCombo_valueVE.set('')

                    
    '''This method allows the user to delete an event from the database. An error is displayed if an event
       hasn't been selected'''
    def deleteEvent(self):
        #display an error if the user hasn't selected an event to delete
        if self.currentEventsListbox.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='Please ensure you have selected an event to delete.')
        else:
            #confirmation message
            myMessage = messagebox.askyesno(title='Delete Event',message='Are you sure you wish to delete your selection?')
            if myMessage == 1:
                #get the details for the event to delete
                self.eventIndex = int(self.currentEventsListbox.curselection()[0])
                self.eventNameToDelete = self.eventsFoundSep[self.eventIndex][1]
                self.ageToDelete = self.eventsFoundSep[self.eventIndex][2]
                self.genderToDelete = self.eventsFoundSep[self.eventIndex][3]
                self.cupCompToDelete = self.eventsFoundSep[self.eventIndex][6]
                #reset the events listbox
                self.currentEventsListbox.delete(END,self.eventIndex)
                #delete the event
                self.removeEvent(self.eventNameToDelete,self.ageToDelete,self.genderToDelete,self.cupCompToDelete)
                #re-search for events from the previous search criteria once an event has been deleted
                if self.ageGroup_valueVE.get() != '':
                    self.searchByAG()
                elif self.gender_valueVE.get() != '':
                    self.searchByG()
                elif self.eventType_valueVE.get() != '':
                    self.searchByEType()
                elif self.cupCombo_valueVE.get() != '':
                    self.searchByCComp()
                elif self.ageGroup_valueVE.get() != '' and self.gender_valueVE.get() != '' and self.eventType_valueVE.get() != '' and self.cupCombo_valueVE.get() != '':
                    self.searchByAll()




                    
class ManageDetails(Toplevel,mainWindow):
    def __init__(self):
        #Open database
        database_opts.__init__(self)
        #Create string variables for entry boxes
        self.houseName = StringVar()
        self.entryAgeVal = StringVar()
        self.entryCupCompVal = StringVar()


    def configManageDetailsScreen(self,master):
        #Make current window 'TopLevel'
        Toplevel.__init__(self)

        #Define window
        self.master = master
        self.title('Manage Details Screen')
        self.geometry('600x400+200+20')
        self.resizable(0,0)
        self.master.configure(background=self.rgb_light_green)
        
        #Notebook widget on window
        self.note = ttk.Notebook(self, width=600,height=400)
        self.ageGroupTab = Frame(self.note,bg=self.rgb_light_green)
        self.cupCompTab = Frame(self.note,bg=self.rgb_light_green)
        self.housesTab = Frame(self.note,bg=self.rgb_light_green)
        self.settingsTab = Frame(self.note,bg=self.rgb_light_green)
        #Add tabs to notebook widget
        self.note.add(self.housesTab, text='Houses')
        self.note.add(self.ageGroupTab, text='Age Groups')
        self.note.add(self.cupCompTab, text='Cup Comps')
        self.note.add(self.settingsTab, text='Settings')
        self.note.pack()
        

        #Houses Window
        #Frames
        self.mhTop = Frame(self.housesTab,bg=self.rgb_light_green,pady=10)
        self.mhTop.pack(side=TOP)
        self.mhLeft = Frame(self.housesTab,bg=self.rgb_light_green)
        self.mhLeft.pack(side=LEFT)
        self.mhRight = Frame(self.housesTab,bg=self.rgb_light_green)
        self.mhRight.pack(side=RIGHT)

        #Top frame - houses title
        self.labelHouses = Label(self.mhTop, text='Manage Houses',bg=self.rgb_light_green,
                                 font=('Courier',24),relief=RAISED).grid(row=0,column=0)

        #Left frame - manage houses 
        self.labelHouseName = Label(self.mhLeft, text='House Name:',
                                    bg=self.rgb_light_green).grid(row=3, column=0,padx=15)
        self.entryHouseName = Entry(self.mhLeft, textvariable=self.houseName).grid(row=3, column=2)
        self.addBut = Button(self.mhLeft, text='Add', command=self.addHouse,
                             highlightbackground=self.rgb_light_green).grid(row=2, column=3)
        self.delBut = Button(self.mhLeft, text='Del', command=self.delHouse,
                             highlightbackground=self.rgb_light_green).grid(row=3, column=3)
        self.clearBut = Button(self.mhLeft, text='Clear', command=self.clearEntry,
                               highlightbackground=self.rgb_light_green).grid(row=4,column=3)

        #Right frame - houses listbox
        self.houseList = Listbox(self.mhRight, width=13, height=10)
        self.houseList.grid(row=1, column=4, rowspan=4, pady=20, padx=20)
        self.fillHouseList()

        
        #Age Categories Window
        #Frames
        self.magTop = Frame(self.ageGroupTab,bg=self.rgb_light_green,pady=10)
        self.magTop.pack(side=TOP)
        self.magLeft = Frame(self.ageGroupTab,bg=self.rgb_light_green)
        self.magLeft.pack(side=LEFT)
        self.magRight = Frame(self.ageGroupTab,bg=self.rgb_light_green)
        self.magRight.pack(side=RIGHT)

        #Top frame - age groups title
        self.labelAgeGroups = Label(self.magTop, text='Manage Age Groups',
                                    bg=self.rgb_light_green,font=('Courier',24),relief=RAISED).grid(row=0,column=0)

        #Left frame - manage age groups
        self.labelAgeGroup = Label(self.magLeft, text='Age Group:',
                                   bg=self.rgb_light_green).grid(row=0,column=0,padx=10)
        self.entryAgeGroup = Entry(self.magLeft, textvariable=self.entryAgeVal).grid(row=0,column=1)
        self.addAgeGroupBut = Button(self.magLeft, text='Add', command=self.addAgeGroup,
                                     highlightbackground=self.rgb_light_green).grid(row=1,column=1)
        self.delAgeGroupBut = Button(self.magLeft, text='Del',command=self.delAgeGroup,
                                     highlightbackground=self.rgb_light_green).grid(row=2,column=1)
        self.clearAgeGroupsBut = Button(self.magLeft, text='Clear',command=self.clearEntry,
                                        highlightbackground=self.rgb_light_green).grid(row=3,column=1)

        #Right frame - age groups listbox
        self.ageGroupListy = Listbox(self.magRight, width=15,height=10)
        self.ageGroupListy.grid(row=0,column=0,padx=60)
        self.fillAgeGroupList()
        

        #Cup Competitions Window
        #Frames
        self.mccTop = Frame(self.cupCompTab,bg=self.rgb_light_green,pady=10)
        self.mccTop.pack(side=TOP)
        self.mccLeft = Frame(self.cupCompTab,bg=self.rgb_light_green)
        self.mccLeft.pack(side=LEFT)
        self.mccRight = Frame(self.cupCompTab,bg=self.rgb_light_green)
        self.mccRight.pack(side=RIGHT)

        #Top frame - cup competitions title
        self.labelCupComps = Label(self.mccTop, text='Manage Competitions',
                                    bg=self.rgb_light_green,font=('Courier',24),relief=RAISED).grid(row=0,column=0)

        #Left frame - manage cup competitions
        self.labelCupComp = Label(self.mccLeft, text='Competition Name:',
                                  bg=self.rgb_light_green).grid(row=0,column=0,padx=10)
        self.entryCupComp = Entry(self.mccLeft, textvariable=self.entryCupCompVal).grid(row=0,column=1)
        self.addCupCompBut = Button(self.mccLeft, text='Add', command=self.addCupComp,
                                    highlightbackground=self.rgb_light_green).grid(row=1,column=1)
        self.delCupCompBut = Button(self.mccLeft, text='Del',command=self.delCupComps,
                                    highlightbackground=self.rgb_light_green).grid(row=2,column=1)
        self.clearCupCompBut = Button(self.mccLeft, text='Clear',command=self.clearEntry,
                                      highlightbackground=self.rgb_light_green).grid(row=3,column=1)

        #Right frame - cup competitions listbox
        self.cupCompListy = Listbox(self.mccRight, width=15,height=10)
        self.cupCompListy.grid(row=4,column=0,padx=20)
        self.fillCupCompList()


        #Settings Window
        #Frames
        self.setTop = Frame(self.settingsTab,bg=self.rgb_light_green,pady=10)
        self.setTop.pack(side=TOP)

        #Top frame - settings 
        self.labelSettings = Label(self.setTop, text='Settings',bg=self.rgb_light_green,font=('Courier',24),relief=RAISED).grid(row=0,column=0,pady=(0,10))
        self.butClearResults = Button(self.setTop, text='Clear Results',highlightbackground=self.rgb_light_green,command=self.clearResults).grid(row=1,column=0,pady=(0,10))
        self.butClearPupils = Button(self.setTop, text='Clear Pupils',highlightbackground=self.rgb_light_green,command=self.clearPupils).grid(row=2,column=0,pady=(0,10))
        self.butClearEvents = Button(self.setTop, text='Clear Events',highlightbackground=self.rgb_light_green,command=self.clearEvents).grid(row=3,column=0,pady=(0,10))
        self.butClearHouses = Button(self.setTop, text='Clear Houses',highlightbackground=self.rgb_light_green,command=self.clearHouses).grid(row=4,column=0,pady=(0,10))
        self.butClearAgeGroups = Button(self.setTop, text='Clear Age Groups',highlightbackground=self.rgb_light_green,command=self.clearAgeGroups).grid(row=5,column=0,pady=(0,10))
        self.butClearCupComps = Button(self.setTop, text='Clear Competitions',highlightbackground=self.rgb_light_green,command=self.clearComps).grid(row=6,column=0,pady=(0,10))
        self.butClearSystem = Button(self.setTop, text='CLEAR ALL',highlightbackground=self.rgb_light_green,command=self.clearAllData).grid(row=7,column=0,pady=(25,0))


    '''This method clears the entry boxes in the manage details section'''
    def clearEntry(self):
        #clear values
        self.houseName.set('')
        self.entryAgeVal.set('')
        self.entryCupCompVal.set('')


    #House Procedures
    '''This method allows the user to add a house into the database and the listbox on the manage houses
       screen. An error is displayed if the user hasn't entered a house'''
    def addHouse(self):
        #displays an error if the the user hasn't entered a house
        if self.houseName.get() == '':
            errormessage = messagebox.showwarning(title='Error',message='Please enter a house to add.')
        else:
            #check for an existing house
            self.checkForRepeats()
            #display an error if house exists
            if self.houseRepeat > 0:
                errormessage = messagebox.showwarning(title='Error',message='House already exists.')
                #reset house entry
                self.houseName.set('')
            else:
                #ensure entry is text only
                try:
                    float(self.houseName.get())
                    self.failed = True
                except ValueError:
                    self.failed = False
                #displays an error if the entry isn't text
                if self.failed == True:
                    errormessage = messagebox.showwarning(title='Error',message='Please ensure your house entry is text only.')
                else:
                    #range check
                    if len(self.houseName.get()) >= 15:
                        checkMessage = messagebox.showinfo(title='Error',message='You have entered a large name. Please ensure this is correct.')
                    #confirmation message
                    myMessage = messagebox.askyesno(title='Add House',message='Are you sure you wish to add %s?' %(self.houseName.get()))
                    if myMessage == 1:
                        #add the house
                        self.insertHouse(self.houseName.get())
                        #refresh the house listbox
                        self.fillHouseList()
                        #reset entry
                        self.houseName.set('')


    '''This method allows the user to delete a house from the listbox and the database once a house has been
       selected. An error is displayed if the user hasn't selected a house to delete'''
    def delHouse(self):
        #displays an error if the user hasn't selected a house
        if self.houseList.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='Please select a house to delete.')
        else:
            #confirmation message
            myMessage = messagebox.askyesno(title='Delete House',message='Are you sure you wish to delete your selection?')
            if myMessage == 1:
                #delete the house
                self.deleteHouse(self.houseList.get(ACTIVE))
                #remove the house from the listbox
                self.houseList.delete(self.houseList.curselection())
                #refresh the house listbox
                self.fillHouseList()


    '''This method fills the house listbox with all the houses from the database'''
    def fillHouseList(self):
        #read in the houses
        self.readInHouses()
        #refresh the house listbox
        for i in self.houseArray:
            self.houseList.delete(END)
        for i in self.houseArray:
            self.houseList.insert(END, i)


    #Age Group procedures
    '''This method allows the user to add an age group into the database and the listbox on the manage age
       groups screen. An error is displayed if the user hasn't entered an age group'''
    def addAgeGroup(self):
        #display an error if the user hasn't entered an age group
        if self.entryAgeVal.get() == '' :
            errormessage = messagebox.showwarning(title='Error',message='Please enter an age group to add.')
        else:
            #check for an existing age group
            self.checkForRepeats()
            #display an error if the age group already exists
            if self.ageRepeat > 0:
                errormessage = messagebox.showwarning(title='Error',message='Age Group already exists.')
                self.entryAgeVal.set('')
            else:
                #range check
                if len(self.entryAgeVal.get()) >= 15:
                    checkMessage = messagebox.showinfo(title='Error',message='You have entered a large name. Please ensure this is correct.')
                #confirmation message
                myMessage = messagebox.askyesno(title='Add Age Group',message='Are you sure you wish to add %s?' %(self.entryAgeVal.get()))
                if myMessage == 1:
                    #add the age group
                    self.insertAgeGroup(self.entryAgeVal.get())
                    #fill the age group listbox
                    self.fillAgeGroupList()
                    #reset the age group entry
                    self.entryAgeVal.set('')
                

    '''This method allows the user to delete an age group from the listbox and the database once a house has been
       selected. An error is displayed if the user hasn't selected a house to delete'''
    def delAgeGroup(self):
        #displays an error if the user hasn't selected an age group
        if self.ageGroupListy.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='Please select an age group to delete.')
        else:
            #confirmation message
            myMessage = messagebox.askyesno(title='Delete Age Group',message='Are you sure you wish to delete your selection?')
            if myMessage == 1:
                #delete an age group
                self.deleteAgeGroup(self.ageGroupListy.get(ACTIVE))
                #delete from the listbox
                self.ageGroupListy.delete(self.ageGroupListy.curselection())
                #fill the age group listbox
                self.fillAgeGroupList()

                
    '''This method fills the age group listbox with all the age groups from the database'''
    def fillAgeGroupList(self):
        #read in the age groups
        self.readInAgeGroups()
        #refresh the age group listbox
        for i in self.ageGroupArray:
            self.ageGroupListy.delete(END)
        for i in self.ageGroupArray:
            self.ageGroupListy.insert(END, i)


    #Cup Competitions Procedures
    '''This method allows the user to add a cup competition into the database and the listbox on the
       manage cup competitions screen. An error is displayed if the user hasn't entered a cup competition'''
    def addCupComp(self):
        #display an error if the user hasn't entered a cup competition
        if self.entryCupCompVal.get() == '' :
            errormessage = messagebox.showwarning(title='Error',message='Please enter a cup competition to add.')
        else:
            #check for an existing cup competition
            self.checkForRepeats()
            #display an error if the cup competition already exists
            if self.cupRepeat > 0:
                errormessage = messagebox.showwarning(title='Error',message='Cup Competition already exists.')
                self.entryCupCompVal.set('')
            else:
                #range check
                if len(self.entryCupCompVal.get()) >= 15:
                    checkMessage = messagebox.showinfo(title='Error',message='You have entered a large name. Please ensure this is correct.')
                #confirmation message
                myMessage = messagebox.askyesno(title='Add Cup Competition',message='Are you sure you wish to add %s?' %(self.entryCupCompVal.get()))
                if myMessage == 1:
                    #add a cup competition
                    self.insertCupComp(self.entryCupCompVal.get())
                    #fill the cup competition listbox
                    self.fillCupCompList()
                    #reset the cup competition entry
                    self.entryCupCompVal.set('')
                          
                    
    '''This method allows the user to delete a cup competition from the listbox and the database once a
       cup competition has been selected. An error is displayed if the user hasn't selected a cup competition
       to delete'''
    def delCupComps(self):
        #displays an error if the user hasn't selected a cup competition
        if self.cupCompListy.curselection() == ():
            errormessage = messagebox.showwarning(title='Error',message='Please select a cup competition to delete.')
        else:
            #confirmation message
            myMessage = messagebox.askyesno(title='Delete Cup Competition',message='Are you sure you wish to delete your selection?')
            if myMessage == 1:
                #delete a cup competition
                self.deleteCupComp(self.cupCompListy.get(ACTIVE))
                #delete from the listbox
                self.cupCompListy.delete(self.cupCompListy.curselection())
                #fill the age group listbox
                self.fillCupCompList()


    '''This method fills the cup competition listbox with all the cup competitions from the database'''
    def fillCupCompList(self):
        #read in the cup competitions
        self.readInCupComps()
        self.cupCompsArray = []
        #refresh the cup competition listbox
        for i in self.cupCompArray:
            self.cupCompListy.delete(END)
        for i in self.cupCompArray:
            self.cupCompsArray.append(i)
        for i in self.cupCompsArray:
            self.cupCompListy.insert(END, i)


    '''This method checks for a repeat house, age group or cup competition from the database'''
    def checkForRepeats(self):
        #check for a repeat house, age group or competition
        self.houseRepeat = 0
        self.ageRepeat = 0
        self.cupRepeat = 0
        #set repeat variable to 1 if there is a repeat
        for i in self.houseArray:
            if i == self.houseName.get():
                self.houseRepeat += 1
        for i in self.ageGroupArray:
            if i == self.entryAgeVal.get():
                self.ageRepeat += 1
        for i in self.cupCompsArray:
            if i == self.entryCupCompVal.get():
                self.cupRepeat += 1
       

    #Settings Procedures
    '''This method asks for confirmation if the the user wants to clear all the results from the database'''
    def clearResults(self):
        myMessage = messagebox.askyesno(title='Clear Results',message='Are you sure you want to clear all the results in the system?')
        if myMessage == 1:
            myMessage2 = messagebox.askyesno(title='Clear Results',message='Are you sure? You cannot undo this.')
            if myMessage2 == 1:
                self.clearResultsTable()
                confirmClear = messagebox.showinfo(title='Clear Results',message='All results have been deleted. Please close this window to save your changes.')


    '''This method asks for confirmation if the the user wants to clear all the pupils and results from
       the database'''
    def clearPupils(self):
        myMessage = messagebox.askyesno(title='Clear Pupils',message='Are you sure you want to clear all the pupils in the system? All their corresponding results will also be erased.')
        if myMessage == 1:
            myMessage2 = messagebox.askyesno(title='Clear Pupils',message='Are you sure? You cannot undo this.')
            if myMessage2 == 1:
                self.clearPupilsTable()
                confirmClear = messagebox.showinfo(title='Clear Pupils',message='All pupils and their results have been deleted. Please close this window to save your changes.')


    '''This method asks for confirmation if the the user wants to clear all the events and results
       from the database'''
    def clearEvents(self):
        myMessage = messagebox.askyesno(title='Clear Events',message='Are you sure you want to clear all the events in the system? All the results will also be cleared.')
        if myMessage == 1:
            myMessage2 = messagebox.askyesno(title='Clear Events',message='Are you sure? You cannot undo this.')
            if myMessage2 == 1:
                self.clearEventsTable()
                confirmClear = messagebox.showinfo(title='Clear Events',message='All events have been deleted. Please close this window to save your changes.')


    '''This method asks for confirmation if the the user wants to clear all the houses from the database'''
    def clearHouses(self):
        myMessage = messagebox.askyesno(title='Clear Houses',message='Are you sure you want to clear all the houses in the system?')
        if myMessage == 1:
            myMessage2 = messagebox.askyesno(title='Clear Houses',message='Are you sure? You cannot undo this.')
            if myMessage2 == 1:
                self.clearHouseTable()
                confirmClear = messagebox.showinfo(title='Clear Houses',message='All houses have been deleted. Please close this window to save your changes.')


    '''This method asks for confirmation if the the user wants to clear all the age groups from the database'''
    def clearAgeGroups(self):
        myMessage = messagebox.askyesno(title='Clear Age Groups',message='Are you sure you want to clear all the age groups in the system?')
        if myMessage == 1:
            myMessage2 = messagebox.askyesno(title='Clear Age Groups',message='Are you sure? You cannot undo this.')
            if myMessage2 == 1:
                self.clearAgeGroupTable()
                confirmClear = messagebox.showinfo(title='Clear Age Groups',message='All age groups have been deleted. Please close this window to save your changes.')


    '''This method asks for confirmation if the the user wants to clear all the cup competitions from the database'''
    def clearComps(self):
        myMessage = messagebox.askyesno(title='Clear Competitions',message='Are you sure you want to clear all the competitions in the system?')
        if myMessage == 1:
            myMessage2 = messagebox.askyesno(title='Clear Competitions',message='Are you sure? You cannot undo this.')
            if myMessage2 == 1:
                self.clearCupCompTable()
                confirmClear = messagebox.showinfo(title='Clear Competitions',message='All competitions have been deleted. Please close this window to save your changes.')


               
### Main Program ###
                
def mainProg():
    #Define window and size
    root = Tk()
    root.resizable(0,0)
    #Create instance of mainWindow on root
    programWindow = mainWindow(root)
    
if __name__ == "__main__":
    #Run program
    mainProg()















