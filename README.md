# RHCSA-Practice-Archive-and-Compress-Files-with-tar
RHCSA Practice — Archive and Compress Files with tar
Today’s focused objective: Create, inspect, extract, compress, and verify archives using tar, gzip, and bzip2.
Estimated time: 45 minutes
Lab system: RHEL 9 or RHEL 10
Privileges required: None
Exam relevance: Red Hat’s current EX200 objectives require candidates to archive, compress, unpack, and uncompress files using tar, gzip, and bzip2. Official RHCSA EX200 objectives
This lesson builds on your earlier file-management, permissions, and link exercises.
1. Archive versus compression — 7 minutes
Archiving and compression are related, but they are not the same thing.
- Archiving combines multiple files and directories into one file.
- Compression attempts to reduce the amount of storage that data consumes.
A plain tar archive usually ends in:
backup.tar

A gzip-compressed tar archive commonly ends in:
backup.tar.gz
backup.tgz

A bzip2-compressed tar archive commonly ends in:
backup.tar.bz2
backup.tbz2

The important tar options
Option	Meaning
-c	Create an archive
-t	List an archive’s contents
-x	Extract an archive
-f	Specify the archive filename
-v	Show files as they are processed
-z	Use gzip compression
-j	Use bzip2 compression
-C	Change to a directory before operating


A useful memory pattern is:
Create:  tar -czf
List:    tar -tzf
Extract: tar -xzf

The f option identifies the archive filename, so the archive’s name immediately follows the options:
tar -czf ARCHIVE.tar.gz SOURCE

2. Prepare the safe lab — 5 minutes
Everything in this lesson stays under /tmp/rhcsa-archive-lab.
Safety warning
The first command recursively removes only an earlier copy of this temporary lab. Carefully verify that the path is exactly /tmp/rhcsa-archive-lab before running it.
rm -rf /tmp/rhcsa-archive-lab
mkdir -p /tmp/rhcsa-archive-lab/application/{config,logs,scripts}
mkdir -p /tmp/rhcsa-archive-lab/{archives,restore}
cd /tmp/rhcsa-archive-lab
pwd

Expected location:
/tmp/rhcsa-archive-lab

Create realistic application data:
printf '%s\n' \
  'application=InventoryApp' \
  'environment=production' \
  'database=db01.example.test' \
  > application/config/application.conf

printf '%s\n' \
  '2026-09-15 18:45:00 InventoryApp started' \
  '2026-09-15 18:45:02 Database connection successful' \
  > application/logs/application.log

printf '%s\n' \
  '#!/bin/bash' \
  'echo "InventoryApp health check: PASS"' \
  > application/scripts/healthcheck.sh

chmod 640 application/config/application.conf
chmod 640 application/logs/application.log
chmod 750 application/scripts/healthcheck.sh

Verify the starting state:
find application -printf '%M %m %p\n'

Important expected permissions:
-rw-r----- 640 application/config/application.conf
-rw-r----- 640 application/logs/application.log
-rwxr-x--- 750 application/scripts/healthcheck.sh

3. Guided practice: create an uncompressed archive — 7 minutes
Create a standard tar archive:
tar -cvf archives/inventory-app.tar application

This means:
- c — create a new archive
- v — display each processed pathname
- f — use the following filename as the archive
- archives/inventory-app.tar — archive being created
- application — directory being archived
Verify that it exists:
ls -lh archives/inventory-app.tar
file archives/inventory-app.tar

Expected file description:
POSIX tar archive

List its contents without extracting anything:
tar -tvf archives/inventory-app.tar

The t operation is very important during the exam. It allows you to inspect an archive before deciding where and how to extract it.
You should see entries resembling:
application/
application/config/
application/config/application.conf
application/logs/
application/logs/application.log
application/scripts/
application/scripts/healthcheck.sh

Notice that the archive contains the top-level application directory. Therefore, extracting it creates an application directory at the destination.
4. Guided practice: gzip compression — 8 minutes
Create a gzip-compressed archive:
tar -czvf archives/inventory-app.tar.gz application

The new option is:
z = compress using gzip

Inspect it:
ls -lh archives/inventory-app.tar archives/inventory-app.tar.gz
file archives/inventory-app.tar.gz

Expected description:
gzip compressed data

The compressed archive will usually be smaller, although very small test data can make the difference less dramatic.
Test whether the gzip stream is valid:
gzip -t archives/inventory-app.tar.gz
echo $?

Expected exit code:
0

In Linux:
- 0 generally means success.
- A nonzero code indicates a problem.
List the compressed archive without extracting it:
tar -tzf archives/inventory-app.tar.gz

5. Guided practice: extract safely — 8 minutes
A common administrative mistake is extracting an unfamiliar archive into the current working directory. Inspect the archive first and extract it into a dedicated destination.
The restore directory already exists and is empty:
ls -la restore

Extract the gzip archive there:
tar -xzvf archives/inventory-app.tar.gz -C restore

This means:
- x — extract
- z — process gzip compression
- v — display extracted files
- f — identify the archive filename
- -C restore — change to restore before extracting
Inspect the result:
find restore -printf '%M %m %p\n'

You should now have:
restore/application/config/application.conf
restore/application/logs/application.log
restore/application/scripts/healthcheck.sh

Confirm the file contents:
cat restore/application/config/application.conf
cat restore/application/logs/application.log
restore/application/scripts/healthcheck.sh

Expected script output:
InventoryApp health check: PASS

Verify that extraction preserved the data
Generate checksums for the original and restored files:
sha256sum application/config/application.conf \
  restore/application/config/application.conf

sha256sum application/logs/application.log \
  restore/application/logs/application.log

sha256sum application/scripts/healthcheck.sh \
  restore/application/scripts/healthcheck.sh

For each pair, the long checksum values should match.
Compare the entire directory trees:
diff -r application restore/application

Expected result:
No output

As you learned previously, no diff output means no differences were found.
Check that the executable permission survived:
stat -c '%A %a %n' \
  application/scripts/healthcheck.sh \
  restore/application/scripts/healthcheck.sh

Both should show:
-rwxr-x--- 750

6. Guided practice: bzip2 compression — 4 minutes
First verify that bzip2 is available:
command -v bzip2

A typical result is:
/usr/bin/bzip2

Create a bzip2-compressed archive:
tar -cjvf archives/inventory-app.tar.bz2 application

The j option selects bzip2 compression.
Inspect and test it:
file archives/inventory-app.tar.bz2
bzip2 -t archives/inventory-app.tar.bz2
echo $?

Expected exit code:
0

List its contents:
tar -tjf archives/inventory-app.tar.bz2

You do not need to memorize every filename extension, but you do need to associate:
gzip  → z → .tar.gz
bzip2 → j → .tar.bz2

7. Independent challenge — 4 minutes
Do not look back at the guided commands while completing this exercise.
Starting state
Remain in:
/tmp/rhcsa-archive-lab

Assignment
Your manager asks you to preserve only the application’s configuration and scripts—not its logs.
1. Create a gzip-compressed archive named:
archives/inventory-config-backup.tar.gz

2. Include only:
application/config
application/scripts

3. Do not include:
application/logs

4. List the archive without extracting it and prove that no log file is present.
5. Create this extraction location:
challenge-restore

6. Extract the archive into that location.
7. Verify:
   - application.conf was restored.
   - healthcheck.sh was restored.
   - application.log was not restored.
   - The restored script is still executable.
   - The restored files match the originals.
8. Independent verification commands
After completing the challenge, use tests like these, substituting the correct paths:
test -f PATH && echo "PASS: file exists"
test ! -e PATH && echo "PASS: unwanted file is absent"
test -x PATH && echo "PASS: script is executable"
diff ORIGINAL RESTORED
tar -tzf ARCHIVE

Your final archive listing should include configuration and script entries, but it should not contain:
application/logs/application.log

9. Knowledge check
Answer these without reviewing the lesson:
1. What is the difference between archiving and compression?
2. Which tar operation creates an archive?
3. Which operation lists contents without extraction?
4. Which operation extracts an archive?
5. Which option tells tar to use gzip?
6. Which option tells tar to use bzip2?
7. Why should you inspect an unfamiliar archive before extracting it?
8. What does -C restore accomplish?
9. What does an exit status of 0 normally mean?
10. How can you verify that restored content matches the original?
11. Which command creates a gzip-compressed archive?
12. Which command lists a bzip2-compressed archive?
I would consider this objective mastered when you can create, inspect, extract, and verify both .tar.gz and .tar.bz2 archives without referring to notes.
Safe cleanup
Safety warning
The following command recursively deletes the complete temporary practice directory. Confirm that the path is exactly /tmp/rhcsa-archive-lab.
cd /tmp
rm -rf /tmp/rhcsa-archive-lab

Verify removal:
test ! -e /tmp/rhcsa-archive-lab \
  && echo "PASS: archive lab removed safely" \
  || echo "CHECK: archive lab still exists"

Expected result:
PASS: archive lab removed safely
