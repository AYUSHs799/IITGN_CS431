# INSTRUCTIONS

## Please read the instructions properly before starting the assignment!

1. Submit to Canvas a "tar" file containing your submission, a PDF file containing your explanations and any codefiles before the due date. Late submissions incur penalties as described in the first lecture (first you use upgrace hours, then you lose points). Note that only PDF submissions will be graded. You must use at most one page for your explanation for each problem (code you write may be on additional pages). Start each problem on a new page.
2. Be neat and concise in your explanations.
3. Feel free to discuss the assignment in general terms with other people, but the answers must be your own. You are encouraged to shared resources (e.g., online resources you found helpful) and discuss homework assignment solutions.
4. Post any clarifications or questions regarding this homework in Canvas Discussions
5. References: List resources outside of class material that helped you solve a problem. This includes online video tutorials, other problems on other platforms, etc. Remember that source code available online (e.g. stackoverflow) also needs to be cited.
 
--- 
# 1. Access Control

You have been assigned a task to set up an assignment submission system using Unix file system. Alice is the instructor, Bob and Charlie are the TAs, and ST1 and ST2 are two students enrolled in this class. Files of interest include assignment handouts, submitted assignments, and grade reports.

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

Implement the submission system according to above policies by creating the directories and setting up proper ACL policies. This assignment requires you to create new Linux groups and users. To avoid messing up with your operating system, we recommend you to work under a Docker container. To set up a Docker container running Ubuntu 20.04, use the following commands: 

```ps
    # Create a directory that is shared with the container for you to put your submission file
    mkdir shared_folder
``` 

```ps
    # Create the container, named acp
    docker run --name acp -d -t \
        -v "$(pwd)"/shared_folder:/shared_folder \
        ubuntu:20.04
```
```ps
    # Log into the container
    docker exec -it acp bash
```
```ps
    # Install acl package
    apt-get update
    apt-get install acl
```
```ps
    # Next, you can work with the assignment
    # TODOs:
    #       Create the groups, users and the directory structure
    #       Set up ACL policies
    #       Package the directory tree as a tar file
    #       Include the writeup in the top-level directory before submitting the tar file
```
```ps
    # After you are done, copy your ACP.tar to the shared folder
    cp ACP.tar /shared_folder
```
```ps
    # Now you can access ACP.tar outside from the container
    # and submit it
    # To destroy the container, first remove the shared files
    # from inside the container to prevent permission issues
    rm /shared_folder/ACP.tar
```
```ps
    # Outside the container
    docker stop acp
    docker rm acp
    rm -r shared_folder
```
Please note that directly using tar to archive files and folders will discard permission information, you need --acls and -p arguments to preserve both ACL and Unix permissions. Also, when giving a user/group the read/write permission on a folder, you may want to give execute permission too, so that shell utils can access the folder.

You need to submit both an archive of the system you’ve implemented and the write-up (missing write-up will result in a 0 score).

Below, we specify the requirements of the two submissions:

* In your ACL submission, please submit a “ACP.tar” file, which is an archive that satisfies the following folder structure (please do not create any files inside the folders):

    ```js
        ACP
        |__ writeup.pdf
        |__ assignments/
            |__ handouts/
            |__ submissions/
                |__ st1/
                |__ st2/
            |__ grade reports/
                |__ st1/
                |__ st2/
    ```

* Please use the uid, gid, and the name given below to create your users and groups, and setup permissions for each folder. The name you use when creating users and groups will affect grading.

    ```js
        group: instructors (gid: 20001)
        |__ user: alice (uid: 10001)
    ```
    ```js
        group: tas (gid: 21001)
        |__ user: bob (uid: 11001)
        |__ user: charlie (uid: 11002)
    ```
    ```js
        group: students (gid:22001)
        |__ user: st1 (uid: 12001)
        |__ user: st2 (uid: 12002)
    ```

### <b> In your write-up submission, describe how you setup the permissions (include the commands you use) and explain your design. </b>