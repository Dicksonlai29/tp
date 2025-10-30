---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# Tuiniverse Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

_{ list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well }_

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams are in this document `docs/diagrams` folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* has a need to manage a significant number of contacts
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: manage contacts faster than a typical mouse/GUI driven app


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                                     | I want to …​                                            | So that I can…​                                                          |
|----------|---------------------------------------------|---------------------------------------------------------|--------------------------------------------------------------------------|
| `* * *`  | new user                                    | see usage instructions                                  | refer to instructions when I forget how to use the App                   |
| `* * *`  | tutor                                       | add a new student                                       |                                                                          |
| `* *`    | tutor                                       | add a new parent                                        | view parent's details                                                    |
| `* *`    | tutor                                       | link a student with their parent's contact              | bill the parent and contact them regarding their child’s academic matter |
| `* * *`  | tutor                                       | delete a person                                         | remove entries that I no longer need                                     |
| `* * *`  | tutor                                       | edit a person's information                             | update their information                                                 |
| `* * *`  | tutor                                       | find a person by name                                   | locate details of persons without having to go through the entire list   |
| `* *`    | tutor with many persons in the address book | sort persons by name                                    | locate a person easily                                                   |
| `* *`    | tutor                                       | see students' payment status                            | track who has and has not paid                                           |
| `* *`    | tutor                                       | list students who have not made payment                 | track students who have not paid                                         |
| `* *`    | tutor                                       | view my schedule for today                              | know which locations I will be travelling to today                       |
| `*`      | busy tutor                                  | view upcoming classes for the week                      | plan, prepare and organise lesson materials in advance                   |
| `*`      | tutor who hates commuting long distances    | know the most efficient time slot to add in a student   | minimize commute time                                                    |
| `*`      | tutor who frequently uses the app           | edit a class for a certain week only                    | reschedule a class and without it affecting my regular schedule          |
| `* * *`  | tutor                                       | add a new class                                         | view class details                                                       |
| `* *`    | forgetful tutor                             | be prevented from adding classes of conflicting timings | avoid accidentally holding two classes at the same time                  |
| `* *`    | tutor                                       | see students' bill for the month                        | track each student's bill                                                |
| `* *`    | tutor                                       | mark students' attendance after every class             | track class hours and calculate the student's bill                       |
| `*`      | tutor motivated to see students improve     | add a note for a student before their class             | track class content for the student                                      |
| `*`      | tutor motivated to see students improve     | edit a student's note after a class                     | check student's progress                                                 |
| `*`      | tutor who teaches multiple subjects         | filter students by subject                              | prepare resources and reuse lesson materials for similar classes         |
| `*`      | tutor who teaches multiple subjects         | organise students by subject                            | better plan lesson materials and lesson outlines                         |

*{More to be added}*

### Use cases

---

## **Use Case List**

1. [UC-AddStudent](#uc-addstudent)
2. **Edit a Student** (placeholder for future UC)
3. [UC-AddClass](#uc-addclass)
4. [UC-Pay](#uc-pay)
5. **Find a Student** (placeholder for future UC)
6. [UC-DeleteStudent](#uc-deletestudent)
7. [UC-DeleteClass](#uc-deleteclass)
8. [UC-ListPaid](#uc-listpaid)
9. [UC-ListUnpaid](#uc-listunpaid)
10. [UC-ListOverdue](#uc-listoverdue)
11. [UC-AddAndBillStudent](#uc-addandbillstudent)
12. [UC-QuitStudent](#uc-quitstudent)
13. [UC-SeeSummaryForMonth](#uc-seesummaryformonth)

---

## **UC-AddStudent**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-AddStudent
**Actor:** Tutor

### **MSS**
1. Tutor chooses to add a student.
2. Tutor enters all details in order.
3. TSMS validates the details:
    - Name, address, contact number, and parent fields are non-blank.
    - Contact number must contain only numerical digits.
4. TSMS creates the student and displays a message confirming successful creation.
   **Use case ends.**

### **Extensions**
- **3a.** TSMS detects an error in a field.
    - **3a1.** TSMS returns an error message.
    - **3a2.** TSMS clears user input.
      **Use case ends.**

- **3b.** TSMS detects unknown command markers.
    - **3b1.** TSMS returns an error message.
    - **3b2.** TSMS clears user input.
      **Use case ends.**

---

## **UC-AddClass**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-AddClass
**Actor:** Tutor

### **MSS**
1. User adds a class using the command.
2. TSMS checks that the student index exists.
3. TSMS validates the class details:
    - Subject matches existing subjects.
    - Level matches existing levels.
    - Day is a valid day.
    - Time is valid.
    - Hourly rate is a number.
4. TSMS checks for clashes in the timetable.
5. TSMS creates and saves the class, linking it to the student.
   **Use case ends.**

### **Extensions**
- **1a.** Tutor omits a field.
    - **1a1.** TSMS returns an error message.
    - **1a2.** TSMS clears user input.
      Steps **1a1–1a2** repeat until valid input is entered.
      **Use case ends.**

- **2a.** TSMS cannot find a student with that index.
    - **2a1.** TSMS returns an error message.
    - **2a2.** TSMS clears user input.
      Steps **2a1–2a2** repeat until valid input is entered.
      **Use case ends.**

- **4a.** TSMS detects a timetable clash.
    - **4a1.** TSMS returns an error message.
    - **4a2.** TSMS clears user input.
      **Use case ends.**

---

## **UC-Pay**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-Pay
**Actor:** Tutor

**Guarantees:**
- Student's payment status changes.
- Student shows up in list of paid students.

### **MSS**
1. Tutor chooses a student who has paid.
2. TSMS validates the student index.
3. TSMS verifies that the student has outstanding payment.
4. TSMS updates the student’s payment status to *Paid*.
   **Use case ends.**

### **Extensions**
- **1a.** Tutor omits input.
    - **1a1.** TSMS returns an error message.
    - **1a2.** TSMS clears user input.
      Steps **1a1–1a2** repeat until valid input is entered.
      **Use case ends.**

- **2a.** TSMS cannot find a student with that index.
    - **2a1.** TSMS returns an error message.
    - **2a2.** TSMS clears user input.
      Steps **2a1–2a2** repeat until valid input is entered.
      **Use case ends.**

- **3a.** Student has no outstanding payment.
    - **3a1.** TSMS returns an error message.
      **Use case ends.**

---

## **UC-DeleteStudent**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-DeleteStudent
**Actor:** Tutor

### **MSS**
1. Tutor chooses to delete a student.
2. TSMS validates whether the student exists.
3. TSMS displays a confirmation prompt.
4. Tutor confirms the deletion.
5. TSMS deletes the student and all associated data (e.g., classes).
   **Use case ends.**

### **Extensions**
- **1a.** Tutor omits the student index.
    - **1a1.** TSMS returns an error message.
    - **1a2.** TSMS clears user input.
      Steps **1a1–1a2** repeat until valid input is entered.
      **Use case ends.**

- **2a.** TSMS cannot find a student with that index.
    - **2a1.** TSMS returns an error message.
    - **2a2.** TSMS clears user input.
      Steps **2a1–2a2** repeat until valid input is entered.
      **Use case ends.**

- **3a.** Tutor chooses not to confirm deletion.
    - **3a1.** TSMS closes the prompt.
    - **3a2.** TSMS clears user input.
      **Use case ends.**

---

## **UC-DeleteClass**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-DeleteClass
**Actor:** Tutor

### **MSS**
1. User deletes a class with the command.
2. TSMS checks that the student index exists.
3. TSMS removes the class from the timetable and unlinks it from the student.
   **Use case ends.**

### **Extensions**
- **1a.** Tutor omits a field.
    - **1a1.** TSMS returns an error message.
    - **1a2.** TSMS clears user input.
      Steps **1a1–1a2** repeat until valid input is entered.
      **Use case ends.**

- **2a.** TSMS cannot find a student with that index.
    - **2a1.** TSMS returns an error message.
    - **2a2.** TSMS clears user input.
      Steps **2a1–2a2** repeat until valid input is entered.
      **Use case ends.**

---

## **UC-ListPaid**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-ListPaid
**Actor:** Tutor

**Guarantees:**
- Lists students who have paid for the current month.
- Lists corresponding amounts paid by each student.

### **MSS**
1. Tutor chooses to list paid students.
2. TSMS retrieves all students marked *Paid* for the current month.
3. TSMS displays the list of students and their amounts.
   **Use case ends.**

### **Extensions**
- **2a.** No students have paid for the month.
    - **2a1.** TSMS displays a message in the console:
  **Use case ends.**

---

## **UC-ListUnpaid**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-ListUnpaid
**Actor:** Tutor

**Guarantees:**
- Lists students who have not paid for the current month.
- Lists corresponding amount owed by each student.

### **MSS**
1. Tutor chooses to list unpaid students.
2. TSMS retrieves all students not marked *Paid* for the current month.
3. TSMS displays the list of students and their owed amounts.
   **Use case ends.**

### **Extensions**
- **2a.** All students have paid.
    - **2a1.** TSMS displays a message in the console.
      **Use case ends.**

---

## **UC-ListOverdue**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-ListOverdue
**Actor:** Tutor

**Guarantees:**
- Lists students who have not paid for previous months.
- Lists corresponding amounts owed.

### **MSS**
1. Tutor chooses to list overdue students.
2. TSMS retrieves all students who have unpaid fees from past months.
3. TSMS displays the list of overdue students and their owed amounts.
   **Use case ends.**

### **Extensions**
- **2a.** All students have paid for previous months.
    - **2a1.** TSMS displays a message in the console.
      **Use case ends.**

---

## **UC-AddAndBillStudent**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-AddAndBillStudent
**Actor:** Tutor, Student, Parent

### **Preconditions**
- No existing student with the same details.
- No class schedule clashes.

### **MSS**
1. Tutor takes in a new student.
2. Student confirms class details with the tutor.
3. Parent pays the student’s fees for the month.
4. Tutor <ins>adds a student (UC-AddStudent)</ins>.
5. Tutor <ins>adds a parent (UC-AddParent)</ins>.
6. Tutor <ins>adds a class for the student (UC-AddClass)</ins>.
7. Tutor <ins>marks payment received (UC-Pay)</ins>.
   **Use case ends.**

---

## **UC-QuitStudent**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-QuitStudent
**Actor:** Tutor, Student

### **Precondition**
- Student currently has classes with the tutor.

### **MSS**
1. Student terminates classes with the tutor.
2. Tutor <ins>deletes the student (UC-DeleteStudent)</ins>.
   **Use case ends.**

### **Extensions**
- **2b.** Student has unpaid fees.
    - **2b1.** TSMS displays an additional confirmation prompt.
    - **2b2.** Tutor confirms the deletion.
    - **2b3.** TSMS updates owed amount and total earnings.
      **Use case ends.**

---

## **UC-SeeSummaryForMonth**

**System:** Tuiniverse Student Management System (TSMS)
**Use case:** UC-SeeSummaryForMonth
**Actor:** Tutor

### **MSS**
1. Tutor <ins>views list of paid students (UC-ListPaid)</ins>.
2. Tutor <ins>views list of unpaid students (UC-ListUnpaid)</ins>.
3. Tutor <ins>views list of overdue students (UC-ListOverdue)</ins>.
   **Use case ends.**

---

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2. The app should be able to accomodate up to 1000 tuition teachers concurrently without any performance compromises
3. A tutor with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4. User actions should take less than 2 seconds
5. The system should include comprehensive unit and integration tests with ≥ 75% code coverage.
6. The software shall be fully functional without requiring installation; it should run as a standalone executable (e.g., via java -jar Tuiniverse.jar).

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **GUI**: Graphical User Interface
* **CLI**: Command Line Interface
* **Person**: Student or parent
* **Payment Status**:
  * Paid - The student has paid within the month
  * Unpaid - Payment has been requested less than 2 weeks ago but has not been paid
  * Overdue - Payment has been requested more than 2 weeks ago but has not been paid
* **Bill**: The payment amount owed by a student
* **Subjects**: Math, English, Physics, Chemistry, Biology, Geography, History, Mother tongue, Social Studies, Literature
* **Note**: A comment located in a student's information
* **Schedule**: A timetable for classes containing the time, location, subject of the class and the student taking the class

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
