Đây là một bài về OS command injection 
<img width="1190" height="726" alt="Screenshot 2025-11-09 102332" src="https://github.com/user-attachments/assets/5f7911bd-fafe-42d8-82dd-e9f475a32efd" />

<img width="1093" height="731" alt="Screenshot 2025-11-11 114709" src="https://github.com/user-attachments/assets/696ccf07-0aca-4e3e-a11e-f4df5268df37" />
<img width="1161" height="638" alt="Screenshot 2025-11-11 114740" src="https://github.com/user-attachments/assets/7ea355b5-5704-4249-a1e8-0e9666282807" />

Kết luận là dưới 5 kí tự thì câu lệnh mới chạy được .Ở đây e sẽ dùng lệnh od dùng để hiển thị nội dung
nhị phân của file ở dạng mã số, em dùng lệnh od *: dump toàn bộ nội dung của tất cả file trong thư
mục hiện tại.
<img width="888" height="544" alt="Screenshot 2025-11-09 165339" src="https://github.com/user-attachments/assets/559905c7-17bf-418f-87bc-e4606efe00c6" />

<img width="962" height="923" alt="Screenshot 2025-11-11 120143" src="https://github.com/user-attachments/assets/4b6954c5-bc0b-4253-9869-fc76dabcebae" />
<img width="480" height="133" alt="Screenshot 2025-11-11 120219" src="https://github.com/user-attachments/assets/fbbf1b50-2a11-467a-8e3b-f1bb15d97e55" />

Dữ liệu od trong ảnh là các word (16-bit) ở dạng octal và máy là little-endian, nên khi chuyển từng
word từ octal → 2 byte (little-endian) rồi decode sang ASCII thu đc flag
Script để decode
