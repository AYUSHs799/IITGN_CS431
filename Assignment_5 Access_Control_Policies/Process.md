# Access Control Policies

We have been assigned a task to set up an assignment submission system using Unix file system. Alice is the instructor, Bob and Charlie are the TAs, and ST1 and ST2 are two students enrolled in this class. Files of interest include assignment handouts, submitted assignments, and grade reports.  
   
The desired access control policies are as follows:
* Only instructors and TAs are allowed to write to the assignment handouts.
* All instructors, TAs, and students can read the assignment handouts.
* Instructors and TAs can read students’ assignment submissions.
* A student can read his or her own assignment submissions, but not other students’ submissions.
* A student can write (update) his or her own assignment submissions, but not other students’ assignment submissions.
* Instructors and TAs are not allowed to write to students’ assignment submissions.
* Instructors can read and write the grade reports.
* TAs can read the grade reports, but not write any grade reports.
* A student is allowed to read his or her own grade report, but not other students’ grade reports.
* A student is not allowed to write any grade reports.
* During the semester, a TA can be fired. If that happens, the TA user account will be removed from the TA group and lose any permissions when accessing the file system.

## <li> Initial commands to create the Docker container.
```ps 
    mkdir shared_folder                                     # Making a shared folder to share between host and container.
    docker run --name acp -d -t \   
                -v "$(pwd)"/shared_folder:/shared_folder \
                ubuntu:20.04                                # Creating the container.
```

## <li> Starting and login to the container.
```ps 
    docker start acp            # Starting the container
    docker exec -it acp bash    # Login to the container
``` 

## <li>Installing acl packages and updating the container.
```ps
    apt-get update          # Updating the container#  
    apt-get install acl     # Installing acl packages
```

## <li>Making Directory structure of ACP.
```ps
    mkdir ACP                                       # Making the root directory
    cd ACP                                          # Navigating to the root directory
    mkdir assignments                               # Making the assignments directory
    cd assignments/                                 # Navigating to the assignments directory
    mkdir 'handouts'                                # Making the handouts directory
    mkdir 'submissions'                             # Making the submissions directory
    mkdir 'grade reports'                           # Making the grade reports directory
    mkdir submissions/st1 submissions/st2           # Creating directories for students submissions
    mkdir grade\ reports/st1 grade\ reports/st2     # Creating directories for students grade reports
```

## <li>Viewing directory structure using tree command.
```ps
sudo apt-get install tree   # Installing tree package
tree assignments            # Viewing the directory structure
```
Below is the output of the tree command.

![image](directory%20structure.png)

All the directories are created as per the requirement. Now we will set the access control policies.

## <li>Creating users and groups.
```ps
    groupadd -g 20001 instructors           # Creating groups for instructors
    groupadd -g 21001 tas                   # Creating groups for TAs
    groupadd -g 22001 students              # Creating groups for students

    useradd -g instructors -u 10001 alice   # Creating users for instructor - Alice
    useradd -g tas -u 11001 bob             # Creating users for TA - Bob
    useradd -g tas -u 11002 charlie         # Creating users for TA - Charlie
    useradd -g students -u 12001 st1        # Creating users for student - ST1
    useradd -g students -u 12002 st2        # Creating users for student - ST2
```

## <li>Setting permissions for assignments directory
```ps
    setfacl -m g:instructors:rx /ACP/assignments/   # Instructors can navigate to the assignments directory
    setfacl -m g:tas:rx /ACP/assignments/           # TAs can navigate to the assignments directory
    setfacl -m g:students:rx /ACP/assignments/      # Students can navigate to the assignments directory

    setfacl -m o::r /ACP/assignments/               # This will ensure anyone else is not able to access the directory.
```

## <li>Setting the permissions to the handout’s directory:-
Only instructors and TAs are allowed to write the assignment handouts, and all instructors, TAs, and students can read the assignment handouts.

```ps
    setfacl -m g:instructors:rwx /ACP/assignments/handouts  # Instructor can read and write to the handouts directory
    setfacl -m g:tas:rwx /ACP/assignments/handouts          # TAs can read and write to the handouts directory
    setfacl -m g:students:rx /ACP/assignments/handouts      # students can only read the handouts directory
    
    setfacl -m o::r /ACP/assignments/handouts               # This will ensure anyone else is not able to access the directory.
```

## <li> Setting the permissions to the submission’s directory:-

Instructors and TAs can read students’ assignment submissions. A student can read his or her 
own assignment submissions but not other students’ submissions. A student can write (update) 
his or her own assignment submission but not other students’ assignment submissions.
Instructors and TAs are not allowed to write to students’ assignment submissions

```ps
    Setfacl -m g:instructors:rx /ACP/assignments/submissions    # Instructors can read students’ assignment submissions.
    setfacl -m g:tas:rx /ACP/assignments/submissions            # TAs can read students’ assignment submissions.
    setfacl -m g:students:rx /ACP/assignments/submissions       # Students can navigate to submissions folder but cannot create a new directory within it.
    
    setfacl -m o::r /ACP/assignments/submissions                # This will ensure anyone else is not able to access the directory.
```
### <ul><ul><li> Not allowing students to access each other’s directory:-
```ps
    setfacl -m u:st1:rwx /ACP/assignments/submissions/st1       # st1 can read and write to his own directory
    setfacl -m u:st1:--- /ACP/assignments/submissions/st2       # st1 cannot access st2’s directory
    
    setfacl -m u:st2:--- /ACP/assignments/submissions/st1       # st2 cannot access st1’s directory
    setfacl -m u:st2/rwx /ACP/assignments/submissions/st2       # st2 can read and write to his own directory
```
## <li>Setting the permissions to grade reports directory:-

Instructors can read and write the grade reports. TAs can read the grade reports but not write 
any grade reports. A student is allowed to read his or her own grade report but not other students’ grade reports. A student is not allowed to write any grade reports.

```ps
    setfacl -m g:instructors:rwx /ACP/assignments/grade\reports # Instructors can read and write the grade reports.
    setfacl -m g:tas:rx /ACP/assignments/grade\reports          # TAs can read the grade reports but not write any grade reports.
    
    setfacl -m g:students:rx /ACP/assignments/grade\reports     # Students can read their own grade reports but not other students’ grade reports.
    setfacl -m o::r /ACP/assignments/grade\reports              # This will ensure anyone else is not able to access the directory.
```
### <ul><ul><li>Only the instructor can write into the student’s directory:-
```ps
    setfacl -m g:instructors:rwx /ACP/assignments/grade\reports/st1     # Instructors can read and write the grade reports.
    setfacl -m g:instructors:rwx /ACP/assignments/grade\reports/st2     # Instructors can read and write the grade reports.
```

### <ul><ul><li> Not allowing students to access each other’s directory:-
```ps  
    setfacl -m u:st1:rx /ACP/assignments/grade\reports/st1      # st1 can read his own grade report
    setfacl -m u:st2:--- /ACP/assignments/grade\reports/st1     # st2 cannot access st1’s directory
    
    setfacl -m u:st1:--- /ACP/assignments/grade\reports/st2     # st1 cannot access st2’s directory
    setfacl -m u:st2:rx /ACP/assignments/grade\reports/st2      # st2 can read his own grade report
```

# References.

* https://www.tecmint.com/secure-files-using-acls-in-linux/ 
* https://www.ibm.com/docs/en/zos/2.3.0?topic=scd-setfacl-set-remove-change-access-control-lists-acls
*  https://www.geeksforgeeks.org/access-control-listsacl-linux/
*  https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-access_control_lists