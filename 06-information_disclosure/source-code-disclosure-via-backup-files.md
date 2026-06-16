Link: https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-via-backup-files

Icon: vial
Source code disclosure via backup files
Description
This lab leaks its source code through a backup file left in a hidden directory. To solve it, find the leaked source code and submit the database password, which is hard-coded inside it.
Solution
<img width="1081" height="1820" alt="image" src="https://github.com/user-attachments/assets/948db7eb-6c80-4444-90c6-ea613e480852" />

I started by reading the page source with CTRL+U. Nothing useful there.
Next I checked /robots.txt — the file a site uses to tell search engines which folders to skip. It listed a hidden directory: backup.
<img width="1168" height="1099" alt="image" src="https://github.com/user-attachments/assets/75f015cf-343a-4261-840f-2576e5790cf3" />

Inside that directory was a backup file: ProductTemplate.java.bak. It held a Java package with the database connection details — including the password, which is the answer needed to solve the lab.


<img width="1103" height="1085" alt="image" src="https://github.com/user-attachments/assets/c6ae1356-bb33-4dbe-b8f0-18e18a46c732" />



<img width="1063" height="1817" alt="image" src="https://github.com/user-attachments/assets/5dd8ac0f-8ed8-459f-937f-96f857cea2d6" />

