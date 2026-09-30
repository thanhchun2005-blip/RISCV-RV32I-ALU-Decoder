# ALU Control Decoder - RISC-V RV32I

[![Language](https://img.shields.io/badge/Language-SystemVerilog-blue.svg)](https://en.wikipedia.org/wiki/SystemVerilog)
[![Standard](https://img.shields.io/badge/Standard-IEEE%201800--2012%2F2017-brightgreen.svg)]()
[![Target ISA](https://img.shields.io/badge/ISA-RISC--V%20RV32I-red.svg)](https://riscv.org/)
[![Tool](https://img.shields.io/badge/Verified%20with-Vivado%202022.2-orange.svg)]()
[![License](https://img.shields.io/badge/License-MIT-green.svg)]()

Khối **ALU Decoder** là một thành phần thiết yếu của Bộ điều khiển trung tâm (**Control Unit**) trong kiến trúc RISC-V RV32I. Nhiệm vụ của nó là kết hợp tín hiệu `ALUOp` (từ Main Decoder) với các trường mã hóa chi tiết trong lệnh (`funct3`, `funct7[5]`, `opcode[5]`) để sinh ra mã điều khiển `ALUControl` 4-bit chính xác cho khối ALU.

---

## 📌 Nguyên lý Hoạt động & Bảng Giải mã

```
                    +----------------------+
  op_5 ------------>|                      |
  funct3 [2:0] ---->|     alu_decoder      |-----> ALUControl [3:0]
  funct7_5 -------->|    (Combinational)   |
  ALUOp [1:0] ----->|                      |
                    +----------------------+
```

### 📋 Bảng Chân trị Giải mã Chi tiết

| `ALUOp[1:0]` | `op_5` | `funct3` | `funct7_5` | Lệnh RISC-V | `ALUControl[3:0]` | Tên phép toán |
| :---: | :---: | :---: | :---: | :--- | :---: | :--- |
| `2'b00` | X | X | X | `LW`, `SW`, `AUIPC` | `4'b0000` | ADD (Tính địa chỉ) |
| `2'b01` | X | X | X | `BEQ`, `BNE`, `BLT`, `BGE` | `4'b0001` | SUB (So sánh nhánh) |
| `2'b11` | X | X | X | `LUI` | `4'b0011` | OR (Nạp tức thời) |
| `2'b10` | `0` | `3'b000` | X | `ADDI` | `4'b0000` | ADD |
| `2'b10` | `1` | `3'b000` | `0` | `ADD` | `4'b0000` | ADD |
| `2'b10` | `1` | `3'b000` | `1` | `SUB` | `4'b0001` | SUB |
| `2'b10` | X | `3'b001` | X | `SLL`, `SLLI` | `4'b0101` | SLL |
| `2'b10` | X | `3'b010` | X | `SLT`, `SLTI` | `4'b1000` | SLT |
| `2'b10` | X | `3'b011` | X | `SLTU`, `SLTIU` | `4'b1001` | SLTU |
| `2'b10` | X | `3'b100` | X | `XOR`, `XORI` | `4'b0100` | XOR |
| `2'b10` | X | `3'b101` | `0` | `SRL`, `SRLI` | `4'b0110` | SRL |
| `2'b10` | X | `3'b101` | `1` | `SRA`, `SRAI` | `4'b0111` | SRA |
| `2'b10` | X | `3'b110` | X | `OR`, `ORI` | `4'b0011` | OR |
| `2'b10` | X | `3'b111` | X | `AND`, `ANDI` | `4'b0010` | AND |

> **Điểm mấu chốt thiết kế:** Bit `op_5` (`opcode[5]`) phân biệt lệnh R-type (`op_5 = 1`) và lệnh I-type (`op_5 = 0`). Nhờ đó, lệnh `ADDI` không bao giờ bị nhầm lẫn thành `SUB` kể cả khi các bit trường trên có giá trị tương tự.

---

## 🔌 Đặc tả Cổng Giao tiếp

| Tên cổng | Hướng (Direction) | Độ rộng bit | Ý nghĩa |
| :--- | :---: | :---: | :--- |
| `op_5` | Input | `1` | Bit thứ 5 của opcode (`instr[5]`) |
| `funct3` | Input | `[2:0]` | Trường chức năng 3-bit (`instr[14:12]`) |
| `funct7_5` | Input | `1` | Bit thứ 5 của funct7 (`instr[30]`) |
| `ALUOp` | Input | `[1:0]` | Mã điều khiển từ Main Decoder |
| `ALUControl` | Output | `[3:0]` | Mã thao tác 4-bit cấp cho ALU |

---

## 🧪 Kiểm chứng & Mô phỏng (Verification)

Testbench `testbench/tb_alu_decoder.sv` kiểm tra toàn bộ các nhánh điều kiện bao gồm R-type, I-type, Branch, Load/Store, LUI với cơ chế tự động hiển thị PASS/FAIL.

### Lệnh chạy mô phỏng:

```bash
xvlog -sv rtl/alu_decoder.sv testbench/tb_alu_decoder.sv
xelab alu_decoder_tb -s alu_dec_sim
xsim alu_dec_sim -R
```

---

## 📂 Cấu trúc Thư mục Repo

```
.
├── rtl/
│   └── alu_decoder.sv     # RTL ALU Decoder
├── testbench/
│   └── tb_alu_decoder.sv  # Self-checking testbench
├── .gitignore
└── README.md
```

---

## 👨‍💻 Thông tin Tác giả & Đồ án

- **Sinh viên thực hiện:** Nguyễn Thành Trung
- **Học phần:** Đồ án Môn học 2 (Capstone Project II) – Ngành Kỹ thuật Máy tính
- **Tên đề tài:** Thiết kế, kiểm chứng và triển khai FPGA lõi vi xử lý RISC-V RV32I 32-bit pipeline 5 tầng ở mức RTL bằng SystemVerilog
- **GitHub cá nhân:** [@thanhchun2005-blip](https://github.com/thanhchun2005-blip)
- **Email:** thanhchun2005@gmail.com
