# 3. File-System Interface

## ภาพรวม
บทนี้อธิบายมุมมองที่ผู้ใช้และโปรแกรมมีต่อ file system ได้แก่ ความหมายของ file, file attributes, วิธีเข้าถึงข้อมูล, directory structure, การ mount, การแชร์ไฟล์ และการป้องกันสิทธิ์

## 1) File Concept
- file คือหน่วยเก็บข้อมูลเชิงตรรกะ (`logical storage unit`)
- มองเป็นลำดับข้อมูลต่อเนื่อง
- เนื้อหาและชนิดขึ้นกับผู้สร้าง เช่น text, source, executable, binary

แนวคิดสำคัญ:
- file เป็น `abstract data type`
- ผู้ใช้สนใจชื่อและการใช้งาน มากกว่าตำแหน่งจริงบน disk

## 2) File Attributes
คุณสมบัติที่ file system มักเก็บ เช่น

- name
- identifier
- type
- location
- size
- protection
- time / date / user identification

ข้อมูลเหล่านี้มักถูกเก็บใน `directory structure`

## 3) File Operations พื้นฐาน
การทำงานที่พบบ่อย:
- create
- open
- read
- write
- seek / reposition
- close
- delete
- truncate

จุดสำคัญ:
- `open()` มักย้าย metadata ที่จำเป็นมาไว้ใน memory
- `close()` ใช้ sync กลับโครงสร้างบน disk เมื่อจำเป็น

## 4) File Structure และ Access Methods
โครงสร้างข้อมูลใน file อาจเป็น:
- stream ของ bytes
- records แบบง่าย
- structures ที่ซับซ้อนขึ้น

วิธีเข้าถึงหลัก:
- `Sequential access`
  อ่าน/เขียนตามลำดับ เหมาะกับงาน stream
- `Direct access`
  เข้าถึง block หรือ record ตามตำแหน่งได้ทันที
- วิธีอื่น เช่น indexed methods

## 5) Directory Structure
directory ใช้เก็บข้อมูลเกี่ยวกับไฟล์และจัดระเบียบ namespace

งานหลักของ directory:
- search
- create
- delete
- list
- rename
- traverse

รูปแบบการจัด directory:
- `Single-level directory`
  ง่าย แต่ชื่อชนกันง่าย
- `Two-level directory`
  แยกตามผู้ใช้
- `Tree-structured directory`
  ยืดหยุ่น ใช้งานจริงบ่อย
- `Acyclic-graph directory`
  รองรับการแชร์ไฟล์/โฟลเดอร์

## 6) Path Name
- `Absolute path` เริ่มจาก root
- `Relative path` อิงจาก current directory

ตัวอย่าง:
- `/mail/notes/os.txt` คือ absolute path
- `notes/os.txt` คือ relative path ถ้าอยู่ใน `/mail`

## 7) File Sharing
ระบบอาจให้หลาย user หรือหลาย process แชร์ไฟล์เดียวกันได้

ประเด็นสำคัญ:
- aliasing หรือหลายชื่อชี้ไปหาไฟล์เดียวกัน
- การแชร์ subdirectory
- ความสม่ำเสมอของข้อมูลเมื่อมีหลายคนใช้งานพร้อมกัน

## 8) Mounting
- file system ต้องถูก `mount` ก่อนจึงเข้าถึงได้
- การ mount คือการนำ volume หรือ partition มาผูกเข้ากับ `mount point`
- หลัง mount แล้ว ผู้ใช้จะเข้าถึงมันผ่าน tree เดียวกันของระบบไฟล์

## 9) Protection
การป้องกันสิทธิ์มีไว้ควบคุมว่าใครทำอะไรกับไฟล์ได้บ้าง

สิทธิ์ที่พบบ่อย:
- read
- write
- execute

ใน Unix/Linux มักอิง 3 กลุ่ม:
- owner
- group
- others

## 10) ใจความสำคัญของบทนี้
- file เป็น abstraction ที่ทำให้จัดเก็บข้อมูลบน disk ได้สะดวก
- directory เป็นตัวจัด namespace และ metadata
- access method มีผลกับรูปแบบการใช้งาน
- mounting ทำให้หลาย file systems เชื่อมเป็นต้นไม้เดียว
- protection สำคัญต่อทั้ง security และการใช้งานหลายผู้ใช้

