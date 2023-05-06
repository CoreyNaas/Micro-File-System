# Micro-File-System (MFS) Ideas and Prompt

I got pretty entranced watching [two](https://www.youtube.com/watch?v=qLrTcmyj7Ic) [and](https://www.youtube.com/watch?v=q3_V0EJcD-k) a [half](https://www.youtube.com/watch?v=gKDJLa0OoDc) videos about the NTFS file system, and got started writing my own. Or the plan for one. We'll see if I actually do it!

</span>

# Initial thought that prompted all this

To "move" file1 from folder1 to folder2, _all_ you have to do is three things:

1.  add file1 address to folder2 metadata
2.  add folder2 address to file1 metadata
3.  remove file1's address from folder1's metadata

My only experience with filesystems is some in computer architectures years ago, and a few videos earlier tonight about the linux filsystem hierarchy, and then a few on filesystems in general.

</span>

# Outstanding Questions

## <span style="font-weight:normal;">

*   Folder metadata includes list of file IDs
*   Should this be the address instead?
*   File metadata includes folder ID
*   Should this be the address instead?
*   ID vs address battle can be summed up as indirect vs direct, dynamic vs fast

</span>

# Formal Prompt and Goal

Given a "disk" of an array of BYTEs as address spaces (e.g. "BYTE[4096]"), in your language of choice, create a "filsystem" managing the data content stored in the "disk" using a file and folder table at the beginning of the disk. Folders and files should be stored as metadata entries in the table.

Create a miniature filesystem, which stores files of abitrary lengths and contents in a single address space, intermixed with each other in clusters of a set length. Appending all clusters of a file together will result in the original bit-for-bit data content of the original file.

You should submit the following at each checkpoint for full credit:

*   A plain-english description of what the code should accomplish
*   A bullet point outline of the code's layout
*   A pseudo-code draft of the code
*   The code written in the final language

**Checkpoint 1:** You should be able to:

*   Add and remove folders and files to and from the folder/file table (FFT)
*   Print a list of all folders in the FFT
*   Print a list of all files in a folder
*   Print the metadata of a given folder using its FoID or its name
*   Print the metadata of a given file using its FiID or its name

**Checkpoint 2:** You should be able to:

*   Given a pre-populated address space, read the contents of a given file
*   Move files from one folder to another
*   Delete a file in three ways: Quick, Quick2, and Complete
*   Quick sets the FiMD to "deleted"
*   Quick2 deletes the FiMD entry but doesn't modify the cluster's data content
*   Complete removes all the data content at each cluster and removes the FiMD entry

**Checkpoint 3:** You should be able to:

*   Check if there's enough clusters to add a file, find the largest continuous space if possible
*   Add and remove file data contentfrom the address space

**Checkpoint 4:** You should be able to:

*   Defragment, even if it's cycle-heavy or slow
*   Bonus points if you try different algorithms, or implement them all as options
*   Two possible methods to defragment:
*   Try and move file clusters closer together with the goal of having all files's clusters continuous. the files data contentcan be in any order (i.e., not sorted or by folder). This could a slow in-the-background method. The immediate benefit of files whose clusters are close together is the read speed, since the head doesn't have to move around as far or as often.
*   Start at the bottom and just start scooching the clusters down, updating the fiMD's cluster address list as necessary. since each cluster has its fiID and cluster index at the beginning, this should be straighforward. this will create a single free space at the top of the address space for new data content.

**Checkpoint 5:** You should be able to:

*   Write and use a tiny text editor to at minimum create new text files with data content and store them in the file system. It should be able to print a given text file with either a fiID, a FoID/FiID combo, or a text "/folder/file.txt" path.
*   Nest folders within each other
*   Have shortcuts that exist in one folder to point to a file in another folder

**Checkpoint 6:** You should be able to:

*   Have a MFS-maintained "Lost and Found" where users can restore documents they deleted but want back.
*   This will store the metadata of files that were removed with the "quick" removal option (i.e., the file data content is there until the FiMD entry is removed from the FFT, and not just the flag in the FiMD is set as "deleted") and allow you to "recover" previously deleted files.
*   Be able to "empty" the Lost and Found

**FUTURE:**

*   Use "Clusterbins" to store file data content locations in continuous segments of clusters that help with the speed of reading larger files
*   Be able to modify files with a method that doesn't include deleted and recreating the FFT entry and the clusters. Some sort of compare?
*   Have a list of free cluster locations to speed up locating a file, if currently you're jumping up reading cluster headers until you find a free one, writing data, and saving the address to the FFT entry
*   Checksums because why not?
*   What happens if your file is so large the cluster address exceeds the FFT entry length limit? Clusterbins may help, but maybe you need... FiMDE! The E stand for "Extension", because it acts as a "turn the page" for files with a lot of metadata.
*   This could be a good place to implement FFT defragging that moves FiMDEs to right after their FiMDs.
*   This could also be a good place to implement custom metadata. This might also be where filetypes and their extensions are necessary. The idea being that the name "extension" for the little tiddle at the end of the file's name refers to how the custom metadata for the file required an "File Metadata Extensions" in the MFS!
*   You can store files without extensions, but they won't have any custom metadata, where the custom metadata is stuff like the program to open the file. Without an extension the user is prompted to select a program to open the file into.
*   May also implement speed measure by having a quick table of id-to-address entries, to not have to iterate through all entry headings when looking up based on ID or file name. This would have to be updated as any entry-moving operation is performed.
*   Implement permissions for modifying FFT Entries and by extensions files, or in the common lingo read-write-execute for owner-group-users.

</span>

# Metadata Type Definitions

## Folder Metadata datatype/header entry definition

### Segment 00: Header

*   BYTE[4] Entry beginner
*   BYTE[4] folder ID (foID)
*   BYTE[4] order in folder metadata (FoMD) entries list
*   BYTE[4] total size of sum of files in folder
*   BIT[16] flags
*   0 reserved
*   1 isShortcut
*   2 isDeleted
*   3 isInUse
*   4 isReadyOnly
*   5 isHidden

</span>

### Segment 01: Folder Name

*   BYTE[4] length of name
*   BYTE[length] folder name in ASCII

</span>

### Segment 02: Folder Contents

*   BYTE[4] count of files
*   BYTE[count*4] file IDs
*   BYTE[2] Entry Terminator

</span>

## File Metadata datatype/header entry definition

### Segment 00: Header

*   BYTE[4] Entry Beginner
*   BYTE[4] file ID (FiID)
*   BYTE[4] order in file metadata (FiMD) entries list
*   BYTE[4] total size of sum file in clusters
*   BIT[16] flags
*   0 reserved
*   1 isShortcut
*   2 isDeleted
*   3 isInUse
*   4 isReadyOnly
*   5 isHidden

</span>

### Segment 01: File Name

*   BYTE[4] length of name
*   BYTE[length] file name in ASCII

</span>

### Segment 02: Cluster Addresses

*   BYTE[4] count of clusters
*   BYTE[count*clusterSize] file cluster addresses
*   BYTE[2] Entry Terminator

</span>

## Cluster definition

### Segment 00: Header

*   BYTE[4] fiID
*   BYTE[4] clusterIndex

</span>

### Segment 00: Cluster File Data

*   BYTE[clusterSize] cluster file data content
*   BYTE[4] Cluster Terminator

</span>

# Architecture Design

## The Command Line

Commands and parameters might look like this:


``fs (this just prints the instructions and parameters)``

``fs list (this by itself lists the folder and files in root/no-path)``

``fs list [folder] (this lists the files in the given folder)``

``fs addfolder [folder] (adds folder to root)``

``fs addfile [folder]/[name] (adds file to given folder/path)``

``fs removefolder [folder] (removes folder from root)``

  ``-q will perform a "quick" removal``

  ``-q2 will perform a "quick2" removal``

  ``-c will perform a "complete" removal``

``fs removefile [folder]/[name] (removes file from given folder/path)``

  ``-q will perform a "quick" removal``

  ``-q2 will perform a "quick2" removal``

  ``-c will perform a "complete" removal``

``fs statistics (will show statistics as calculated by the MFS background service)``

``fs movefolder [folder] (will move a folder from its current folder into a given folder)``

``fs movefile [folder] (will move a file from its current folder to a given folder)``

</span>

## The MFS background service must:

*   Run intermittently to
*   Validate and defragment the FFT
*   Perform periodic filesystem partial defragmenting at set intervals
*   Keep track of its defragmenting progress, don't just start from the beginning or a random spot every time.
*   Measure up the total free space in bytes, largest continous free space in bytes
*   Measure up the count of folders and the count of files

</span>

## Parsers

**NOTE:** The basic parsers will just iterate through their respective lists to find the exact file they're given, making this inefficient as it grows (I hear my Data Structures professor talking about Big O notation in the distance...). This could be optimized with a higher-level cache or queue that stores just the connections between files and folder and lives between the file system and service-level operations.

</span>

### The File/Folder Table Parser must:

*   Iterate through the FFT to find a given file or folder's metadata
*   Execute command line commands such as listing files and folders, adding and eleting them, and defragmenting the "disk".

</span>

### The Folder Entry Parser must:

*   Given a folder ID (FoID), iterate through all folder metadata (FoMD) entries and return the data
*   add entries to the end of the FoMD entries segment
*   delete entries from within the FoMD entries segment and close the gap between entries

</span>

### The File Entry Parser must:

*   Given a file ID (FiID), iterate through all file metadata (FiMD) entries and return the data
*   add entries to the end of the FiMD entries segment
*   delete entries from within the FiMD entries segment and close the gap between entries

</span>

### The FFT Defragger (aka adding and removing entries) must:

*   Move data up and down by a given offset
*   Data at the starting offset and ending offset will be deleted. The starting and ending ooffset will be defined at the File/Folder Table (FFT) at index 0 of the disk

</span>

### The FS Defragger (aka moving clusters up and down the address space) must:

*   Move cluster data up and down in the filesystem and update their locations in the FFT entry using the cluster's fiID and cluster index

</span>
