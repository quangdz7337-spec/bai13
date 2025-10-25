# bai13
Dưới đây là file **README.md** hoàn chỉnh và rõ ràng cho dự án STM32 FreeRTOS của bạn 👇

---

#  STM32 FreeRTOS – LED + UART + Mutex + Queue Demo

##  Giới thiệu

Dự án này minh họa cách sử dụng **FreeRTOS** trên vi điều khiển **STM32F103** để:

* Giao tiếp UART (USART1)
* Điều khiển LED theo **tần số và chu kỳ xung (duty cycle)** được gửi từ UART
* Dùng **Queue** để truyền dữ liệu giữa các task
* Dùng **Mutex** để bảo vệ truy cập tài nguyên dùng chung (UART)

---

##  Chức năng chính

###  Cấu trúc hệ thống

Dự án bao gồm 3 task chính:

| Task              | Chức năng                                                                                  | Ưu tiên |
| ----------------- | ------------------------------------------------------------------------------------------ | ------- |
| `LED_Task`        | Nhận dữ liệu tần số & duty cycle từ hàng đợi (queue) và điều khiển LED nhấp nháy tương ứng | 2       |
| `UART_Task`       | Nhận lệnh từ UART, phân tích dữ liệu và gửi vào queue                                      | 2       |
| `UART_Print_Task` | Gửi chuỗi “Hello from Task2” định kỳ để kiểm tra cơ chế **mutex bảo vệ UART**              | 2       |

---



## 📡 Giao tiếp UART

| Thông số     | Giá trị |
| ------------ | ------- |
| Baud rate    | 9600    |
| Data bits    | 8       |
| Stop bits    | 1       |
| Parity       | None    |
| Flow control | None    |

 **TX**: PA9
 **RX**: PA10

Kết nối UART qua USB–UART converter (ví dụ: CP2102, CH340, FTDI).
Mở terminal như **PuTTY**, **RealTerm**, hoặc **TeraTerm** với baudrate 9600.

---

##  LED Task Hoạt Động

Khi nhận được `Signal_t` từ Queue:

```c
typedef struct {
    uint16_t frequency;   // Hz
    uint16_t duty_cycle;  // %
} Signal_t;
```

* Tính chu kỳ `T = 1000 / frequency` (ms)
* LED sáng trong `(T * duty_cycle / 100)` ms, sau đó tắt phần còn lại của chu kỳ
* Lặp lại trong 5 giây trước khi đợi tín hiệu mới

Nếu không có dữ liệu mới trong Queue → LED dùng mặc định:

```
1 Hz, 50% duty cycle
```

---

##  Mutex bảo vệ UART

* UART là tài nguyên dùng chung giữa 2 task (`UART_Task` và `UART_Print_Task`)
* Dùng **xSemaphoreCreateMutex()** để tránh xung đột khi ghi UART

Ví dụ:

```c
if (xSemaphoreTake(xUartMutex, pdMS_TO_TICKS(100)) == pdTRUE) {
    // gửi UART an toàn
    xSemaphoreGive(xUartMutex);
}
```

---

##  FreeRTOS Thành phần chính

* **Queue:** `xSignalQueue` – giao tiếp giữa `UART_Task` và `LED_Task`
* **Mutex:** `xUartMutex` – bảo vệ UART khỏi truy cập đồng thời
* **Task:** 3 task chính (LED, UART, UART_Print)

---

##  Yêu cầu phần cứng

* **MCU:** STM32F103C8T6 (hoặc tương đương)
* **Board:** Blue Pill hoặc Nucleo
* **LED:** Gắn vào chân **PA0**
* **UART–USB Converter:** CP2102 / CH340 / FTDI

---

##  Hướng dẫn build

### 1️ Môi trường

* **IDE:** STM32CubeIDE / Keil uVision / IAR
* **RTOS:** FreeRTOS (được tích hợp sẵn hoặc thêm vào project)
* **Driver:** STM32 Standard Peripheral Library (STM32F10x)

### 2️ Cấu hình FreeRTOS

* `configMINIMAL_STACK_SIZE` ≥ 128
* `configTOTAL_HEAP_SIZE` đủ lớn (>= 4KB)

### 3️ Flash code

* Kết nối ST-Link V2
* Nạp code và mở terminal UART (9600 baud)
* Gửi lệnh `10,70` → LED nhấp nháy 10Hz, duty 70%

---

## 🧾 Tóm tắt các hàm chính

| Hàm                 | Mô tả                                    |
| ------------------- | ---------------------------------------- |
| `GPIO_Config()`     | Cấu hình LED (PA0) dạng push-pull output |
| `USART1_Config()`   | Cấu hình UART1, baudrate 9600            |
| `UART_SendString()` | Gửi chuỗi UART có bảo vệ mutex           |
| `UART_ReadLine()`   | Đọc chuỗi UART đến khi gặp newline       |
| `LED_Task()`        | Điều khiển LED theo tín hiệu từ queue    |
| `UART_Task()`       | Nhận lệnh UART, phân tích và gửi queue   |
| `UART_Print_Task()` | Gửi thông điệp định kỳ qua UART          |

---

## 📊 Ví dụ hoạt động

### ✅ Gửi qua terminal:

```
5,50
```

### 📥 Phản hồi từ board:

```
OK: 5 Hz, 50%
Hello from Task2
Hello from Task2
...
```

### 💡 LED:

Nhấp nháy 5 lần mỗi giây, sáng 50% chu kỳ.

---

## 🧩 Mở rộng đề xuất

* Thêm **command parser** (ex: `LED ON`, `LED OFF`, `STATUS`)
* Ghi log UART vào **SD card** qua SPI
* Dùng **Timer + PWM hardware** thay vì delay mềm
* Thêm **Watchdog task** giám sát hệ thống

---

---

