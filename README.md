# JNachos-Page-Replacement-and-Cleanup
Implementation of Page Replacement Algorithm(FIFO) and Cleaning of Page

Following are the Java files in which changes are made:
(completed the project)

1)Machine.java:

♣	The first step is that I have reduced the number of physical pages to 16 and 200 so that it will result in more number of page faults because of lesser pages. This will result in working of our project for page replacement as intended. 
♣	The other changes are made in the following two functions.
readmem and writemem
(this was also informed by the professor)
	
Following is the change:

(In this change,parameter  “exception” is output of the function that translates virtual addresses to physical addresses.)

if (exception != ExceptionType.NoException)
 {
  raiseException(ExceptionType.AddressErrorException, addr);
     return null;
 }

TO:

if (exception != ExceptionType.NoException) 
 {
     raiseException(exception, addr);
     return null; 
}

2)JNachos.java:

Following are the changes done:
a)	First, I have initialized a FIFO_list ,since I am using the FIFO(simplest) page replacement algorithm for this project. 
It will select the victim page for performing page replacement.

b)	Second, I have used variable processvirtspace_file which is representing a file on disk which has virtual spaces for all processes(programs).

c)	Third, I have used variable (a counter) named pageload_count for keeping information about the next available page in order to write it in processvirtspace_file. This is done by increasing the pageload_count variable once I load the page into processvirtspace_file.

3)TranslationEntry.java:

This file basically define entry either in page table or TLB. I have added a new variable curr_virt_pgno which has the page number of present virtual page in prpcessvirtspace_file. It is not necessary that this page number and Swapping_file page number is same since it is an operating system and has other tasks to do as well. 

4)Cleanup.java:
I have created a new java file for cleanup process. It has an array list  of Page stored it in process_Pagetablelist. It deals with prpcessvirtspace_file cleanup process and make it simple when a process finishes. This file is used in JNachos.java file for further changes.

5) NachosProcess.java:

a)	In this, I have implemented the main memory and processvirtspace_file cleanup process when the process exits. 

b)	The first thing I have done is to check whether the process is using the processvirtspace_file or the main memory since cleanup is necessary for both the storage areas by checking whether the AddrSpace for the process is null. 

c)	In case it is null, then there is no need for the cleanup process. 

d)	In case it is not null, then we have to locate all the pages that are currently there(loaded) in the main memory and for that loop through its page table in order to traverse and check. 

e)	After that, I have set that page to be unused in the BitMap freemap(mFreeMap) showing the page frames which are in use, for every translation entry representing a page that is there in memory, then I have removed that Translation Entry from the FIFO_list so that it will not  be selected as a victim page and then clear that page frame. This way main memory cleanup is implemented.

f)	The next part is to clean the processvirtspace_file in order to complete cleanup process in order to save space.

g)	The first thing is to see whether the process was the last process which was loaded in processvirtspace_file, and this can be done by checking whether the last byte of this process is at same locationas the last byte which is used in file. Then, I have overwritten all the bytes used by this process with 0’s and then decrease the counter pageload_count by the number of pages which are finishing(exits).

h)	But in case the process which is getting exot is not the last procees which got loaded to the processvirtspace_file, then  I have first verified which bytes were loaded and how many are loaded after the process which got exited.

i)	Then, I have read these bytes into the buffer and have write this back to the processvirtspace_file starting from first byte of exiting process. This will delete this process from file.

j)	Now, there will be some unnecessary data as I have overwritten the file starting from starting byte of exiting process. There I have replaced the last byte of this data to the last byte of processvirtspace_file with 0’s.

k)	Next, I have created a separate class cleanup and has Process_Pagetableslist which has references to all the page tables created by processes, I will locate  all the tables whose first page was loaded after the exiting process's first page by  iterating over each page table.Then I have decreased  each Translation Entry's curr_virt_pgno value by the number of pages occupied by the exiting process for all the tables found.

l)	Once this is done, I have decreased the pageload_count by the number of pages occupied by the exiting process. 

m)	Because, all the process which were overwritten shifted the data down by these many pages, I have iterated over each Translation Entry loaded after the exiting program to redirect it's pointer to the correct location. Same way, I have redirected the pageload_count so that it points to the next available page in order to write it in processvirtspace_file.

6) AddrSpace.java:

Following are the changes made:
a)	The first change is done in Addrspace constructor function. The earlier implementation of this constructor used to read an executable file and then it creates page table and will load each page in memory.

b)	Now, because each page was earlier loaded in main memory, when the page table was created, so I have set each physical page number of this translation entry to the next page frame which is available.

c)	Also, setting each entry’s valid bit to true. For this project implementation, I have changed the code and have set the page frame number as -1 which means the page is not there in main memory.

d)	Then set the valid bit to be false, set the curr_virt_pgno to the pageload_count, read the corresponding page from the executable file, and write that page into the processvirtspace_file.

e)	This will load the page from the file to the processvirtspace_file instead of main memory now. This will result in page fault whenever this process runs.

f)	Here, we get to hit the ExceptionHandler.java to fetch the desired page.In order for process cleanup thing, once the page table is created, this table is added to Cleanup.process_Pagetablelist  to help us with the memory/ processvirtspace_file cleanup when the process gets terminated. This detail is mentioned in ExceptionHandler.java file.

g)	Another major change is there in the copy constructor. Here, instead of copying the pages from the main memory which it was doing earlier, pages will be read from processvirtspace_file and then will be copied to the next position which is available in processvirtspace_file.

h)	As a result, the new process has its own copy of the pages in processvirtspace_file. Here, also page table is created as same way as I have explained in constructor above. The only difference is how we load the pages into the processvirtspace_file.

6) ExceptionHandler.java:

Following are the changes done:  (PAGE FAULT Handling- Algorithm: FIFO)

a)	Here, I have implemented the PageFaultException switch case as mentioned in project guidelines. 

b)	The first thing that is need to be done here is to fetch the virtual address of the page which is resulting in page fault.

c)	Then I have checked that if there are any free page frames in the main memory so that I can load the new faulting page into memory, and will read the faulting page from processvirtspace_file into the buffer.

d)	If there exists a free page frame in memory, then I have simply updated the current page's translation entry in the Page Table. Then, updated the physical page number of the page to be the address of the free page frame, and updated the translation entry's valid bit to be True and use bit to be True. 

e)	If in case, there are no free page frames available, then a victim page is need to be located in order to evict. I have simply removed the first Translation Entry form our FIFO list as per the algorithm to implement it.

f)	Now, before I write the faulting page into main memory, I have checked first whether the victim page has been changed as it was loaded into main memory.

g)	If dirty bit of victim page is set, then read that page from main memory and write it back to Swapping_file.

h)	Then set the valid bit of faulting page to be True, use bit to true, and physical page to the physical page of the victim page. 

i)	Next set the dirty bit, valid bit, and use bit of the victim page to False and set the physical page number of the victim page to -1 which means that it is not present main memory.

j)	The last step is to load the faulting page into the free page frame in main memory and add the faulting page's translation entry to the end of our FIFO list as per the FIFO algorithm implementation.

7)SystemCallHandler.java:

I have modified my own project-1 code for fork and join and exit system call in order to perform cleanup for processes. In fork system call, I have changed only one statement since I am adding child processes to the cleanup_processes list now.in case of join, I have added a new condition that join should also return in case, the given process id is greater than the cleanup count. Earlier I was using hashmap but now I am using clean_map which I have declared in cleanup.java file since I am cleaning processes. The same map is used in exit system call as well.

