Download Link: https://assignmentchef.com/product/solved-cs2110-project4-c-programming
<br>
Hello and welcome to Project 4. In this project, you’ll write a C program to interact with a toy file-system of documents.

<h2><a name="_Toc9804"></a>1.1         Background</h2>

For this project, you are given the header file project4.h, and a shell of the C file project4.c. The header file contains the backbone of the toy file-system you will be implementing commands for, as well as three macros which will greatly aid in your implementation for several functions. Keep in mind that you will have to implement these macros.

doc_t doc_system[MAX_NUM_DOCS]; char data[MAX_DOCSIZE * MAX_NUM_DOCS]; uint64_t doc_valid_vector;

// Implement this macro; it should evaluate to a non-zero (true) result if and only if // the “index”th bit is set in doc_valid_vector, which would imply that a doc // is already present at index.

#define GET_DOC_PRESENT(doc_valid_vector, index)

// Implement this macro; it should evaluate to a copy of doc_valid_vector with the // “index”th bit set, showing that a doc is now present at index. #define SET_DOC_PRESENT(doc_valid_vector, index)

// Implement this macro; it should evaluate to a copy of doc_valid_vector with the // “index”th bit cleared, showing that no doc is now present at index. #define CLEAR_DOC_PRESENT(doc_valid_vector, index)

Firstly, we have an array of doc_t structs which represent all of the documents that currently reside in the file-system. However, the doc_t struct does not actually contain the data for a document – all document data is stored in the char array data. This character array is thought of as partitioned into many sections of length MAX_DOCSIZE, where each section may or may not contain a document. The 64-bit integer doc_valid_vector is a bitvector whose <em>n</em>th bit (starting from the LSB) indicates whether or not there is a document at the <em>n</em>th slot of the data array. You will need to update this bit vector as your document system changes.

The header file also defines various constants, the ext_t enum, and the doc_t struct.

typedef enum extension {

DOC = 0,

ASCII = 1 } ext_t;

typedef struct document_t { char name[MAX_NAMESIZE]; ext_t extension; uint8_t permissions; char *data;

int size; int index;

} doc_t;

The extension enum is largely self-explanatory: it indicates that the only file extensions that are valid for documents in our file-system are .doc and .ascii. These extensions do not have any inherent meaning, since all of our documents are simply represented by character data.

The document struct is more involved. It contains its own name and extension (e.g., example.ascii would have the name “example” and the extension ASCII). Furthermore, it contains an 8-bit integer permissions, the first 3 bits of which represent whether or not the user has read, write, and execute permissions for the file. Executing a document has no meaning in this project, but you will have to consider read and write permissions when implementing various commands. The struct also contains a char pointer, which points to the first character of the document in the data array. Finally, a document also contains its own size (i.e. the number of chars in the document) and its position in the data array (note that this index could be calculated from the char pointer, but it is convenient to store it in the struct).

<h1><a name="_Toc9805"></a>2           File-system Commands</h1>

The bulk of your work for this project will be implementing various functions that act on the file-system. The specifications for these functions are listed briefly in the shell code itself, but are explained with more detail in the sections below.

Many of these functions return an integer flag which indicates success or failure. We have provided the macros #define ERROR -1 and #define SUCCESS 0 to help you. Also, any function that receives a pointer as input should error if the pointer is NULL.

Furthermore, we have provided you with some helper functions in helper.c. Feel free to take a look at them to determine their functionality. In the specifications below, we mention any helper functions that will be useful when coding that command. Finally, please do not create your own global variables! Everything you need is given to you.

<h2><a name="_Toc9806"></a>2.1         New</h2>

int new_doc(char* docname);

This function creates a new, empty, document and adds it to the filesystem. It should return −1 if there is no space left in the filesystem for a new document, or if a document with the given name already exists. Otherwise, it should add the new document at the first available slot in the file-system. Files created with this function should have all permissions (i.e. the read, write, and execute bits should all be set to 1). To get the name and extension of the new document, you should use the get_name_ext function that has been implemented for you.

After successfully creating a document, return its index.

<h2><a name="_Toc9807"></a>2.2         Find</h2>

int find_doc(const char* docname);

This function returns the index of the document in the file-system with the given name, returning −1 if such a document does not exist. Once again, you should use the get_name_ext function that has been implemented for you. Once you split the given docname into a name and extension, make sure to compare each of them to each document in the docsystem.

<h2><a name="_Toc9808"></a>2.3         List</h2>

void list(void);

When this function is called, you should print out all valid documents in the file-system (in index order). You should use the print_doc_name function that has been implemented for you.

<h2><a name="_Toc9809"></a>2.4         Append</h2>

int append(doc_t *doc, char *str);

The append function takes a pointer to a document and a string, and expects the string to be appended onto the document. This action is only allowed if the document has write-permissions, and there is enough space for the entire given string to be appended (−1 should be returned if these conditions are not met, and 0 should be returned on success). You should consider using the C functions strlen and strcpy in your implementation.

(Hint: The data for each document in the data array is expected to be zero-terminated – take that into account when deciding whether there is enough space to append the given string. Since MAX_DOCSIZE is the maximum total length of a document’s data, MAX_DOCSIZE – 1 is the maximum size of <em>actual </em>contents, since the zero-terminator will always take up one byte)

<h2><a name="_Toc9810"></a>2.5         Import</h2>

int import(char* docname, FILE *file);

This function takes in a document name (which should be passed to new_doc) and a file-pointer. It should import the contents of the given file into the new document. If the contents of the file are too large to fit into a document, the function should fail (the newly created file should be removed, and you should return −1). The helper function get_file_size will be useful, and you should also use the C function fread in your implementation.

<h2><a name="_Toc9811"></a>2.6         Print and Export</h2>

int print_doc_data(const doc_t *doc); int export(const doc_t *doc, FILE *file);

The print_doc_data function is given to you, and will simply print the entire contents of the given document, followed by a newline. You will have to implement export, which should print the contents to the given file-pointer with fprintf. The function should fail if the given document does not have read access, returning −1. Return 0 on success.

<h2><a name="_Toc9812"></a>2.7         Remove</h2>

int remove_doc(doc_t* doc);

In this function, you should remove the document given as input. Note that you do not need to clear any memory or alter the doc_system array (although you will not be penalized for doing so). You only need to modify the doc_valid_vector.

<h2><a name="_Toc9813"></a>2.8         Change Mode</h2>

int change_mode(doc_t *doc, char* mode_changes);

This function should change the permissions of the given document based on the mode_changes string. This string will have as a first character either + or -, indicating whether we are removing or adding permissions. After this sign, there will be a sequence of at most three characters from the set (r, w, x) indicating which permissions need to be added/removed.

<h1><a name="_Toc9814"></a>3           The Interaction Loop</h1>

After you have completed all of the required functions, you can move onto the interaction loop, where you will implement the interface between the user of your file-system and the commands you wrote. Essentially, the interaction loop repeatedly takes text commands from the user, and performs the requested action. For example, if the user typed new test.doc, you would create a new document called test.doc. The user is repeatedly prompted for these commands until they type exit.

Inside the my_main function, we have already implemented the shell of how this will work, along with the specific implementation of the new and exit commands. To complete this function, you will have to create else if branches for each user command we require, the specifications of which are listed below:

<ol>

 <li>ls: lists all valid documents</li>

 <li>import [external_file] [doc_name]: imports a file</li>

 <li>export [doc_name] [external_file]: exports to a file</li>

 <li>print [doc_name]: prints the contents of a document</li>

 <li>append [doc_name] [string]: appends a string to a document</li>

 <li>chmod [doc_name] [mode_changes]: changes the permissions of a document</li>

</ol>

(Note that we have included the code to parse the user input for you – all you need to do to access the command arguments is use the arguments array, and pass them into functions you defined earlier. Also, you will have to make use of the C functions fopen and fclose for dealing with file input)

<h1><a name="_Toc9815"></a>4           Building &amp; Testing</h1>

All of the commands below should be executed in your Docker container terminal, in the same directory as your project files.

<h2><a name="_Toc9816"></a>4.1         Unit Tests</h2>

To run the autograder locally (without GDB):

// To clean your working directory make clean

// Compile all the required files make tests

// Run the tester object

./tests

This will run all the test cases and print out a percentage, along with details of the <strong>failed test cases</strong>.

Other available commands (after running make tests):

<ul>

 <li>To run a specific test case (to avoid all printing output/debug messages for all test cases):</li>

</ul>

make run-case TEST=testCaseName

<ul>

 <li>To run a test case with gdb:</li>

</ul>

make run-gdb TEST=testCaseName (or no testCase to run all in gdb)

<h2><a name="_Toc9817"></a>4.2         Testing my main</h2>

There are two test cases for the main function that exist in the my main Tests directory. Each consists of a subdirectory with:

<ul>

 <li>An input file (caseX in.txt). This will consist of a list of commands that will be given to your main function, as if you were typing them one after the other on the command line.</li>

 <li>An expected console output (caseX console expected txt), consisting of exactly what the program is expected to print out when given the input file. You must match this exactly.</li>

 <li>For case 2 specifically, an expected export file (case2 export expected txt). This should exactly match the export file produced during the test case.</li>

</ul>

The full program (executable from command line with a working shell) can be used like so:

// To clean your working directory make clean

// Compile all the required files and build the executable make project4

// Run the tester object

./project4

You can then type in commands on the shell to see their effect on the document system.

The autograder for this portion of the assignment is a shell script that will run the test cases against your executable and compare the output text against the expected output. You can run it with the following command. It will show you the differences between your output and the expected output.

make run-main-tests