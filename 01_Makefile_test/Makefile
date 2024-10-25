# Compiler
CC = gcc

# Danh sách các tệp nguồn
src_files = main.c utils.c helper.c test.c

# Lọc các tệp .c và lưu vào biến c_files
c_files = $(filter %.c, $(src_files))

# Danh sách tệp đối tượng tương ứng với các tệp .c
obj_files = $(patsubst %.c, %.o, $(c_files))

# Mục tiêu chính
all: $(obj_files)

# Quy tắc để biên dịch tệp .c thành tệp .o
%.o: %.c
	$(CC) -c $< -o $@

# Mục tiêu để in ra danh sách tệp
print:
	@echo "Tệp nguồn: $(src_files)"
	@echo "Tệp .c: $(c_files)"
	@echo "Tệp đối tượng: $(obj_files)"

# Mục tiêu để dọn dẹp
clean:
	rm -f $(obj_files)
