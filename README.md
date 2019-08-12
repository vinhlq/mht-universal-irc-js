# mht-universal-irc-js

## <a id='irc-docs'></a>1. Tài liệu
  * [Bảng phân bố  irc-command-page-draff](irc-provider-page.md)
  * [Sony irc-command-data-draff](irc-sony-command-data.md)
## <a id='irc-purpose'></a>2. Mục tiêu
* Xây dựng javascript-module cho mô hình **thu thập - quản lý - điều khiển** các thiết bị có giao thức IRC(infrared remote control)

## <a id="irc-defination"></a>3. Định nghĩa
  * IRC: infrared remote control
  * [irc-protocol](#mht-universal-irc-protocol-pubsub)
  * irc-provider: nhà cung cấp thiết bị irc
  * irc-device: thiết bị hỗ trợ irc-protocol
  * irc-client: client điều khiển irc-device
  * universal-irc-device: thiết bị làm nhiệm chuyển tiếp giao thức irc
  * [irc-database](#irc-database): db chứa các dữ liệu về command điều khiển
    * [irc-database-central](#irc-database-central)
    * [irc-database-device](#irc-database-device)

## <a id='universal-irc-device'></a>4. Phân loại thiết bị chuyển tiếp(universal-irc-device)
  * universal-irc-zigbee-device
  * universal-irc-wifi-device

## <a id="irc-command"></a>5. Phân loại command
  * <a id='irc-command-data-user'></a>irc-command-data-user
    * command do user khai báo
    * sử dụng trong nội bộ 1 hệ (gắn với user)
  * <a id='irc-command-data-verified'></a>irc-command-data-verified
    * command được hệ thống xác nhận
    * sử dụng trên toàn hệ thống
    * Phân loại:
      * irc-command-data-verified-raw
        * Là loại command chỉ chứa dữ liệu

      * irc-command-data-verified-script
        * Là loại command có áp dụng script
        * Script nằm dưới [universal-irc-device](#universal-irc-device) được định danh bằng scriptId 
        * Script có thể generate từ client và sử dụng command **irc-command-data-verified-raw**

## <a id="irc-flow"></a>6. Mô tả chu trình
  * Thu thập: learning
    * client wait event ([uirc-executed](#uirc-executed)) -> irc-data -> kiểm tra tồn tại -> **[irc-command-data-user](#irc-command-data-user)** -> lưu [irc-database-central](#irc-database-central)
    * management-client: phân loại **[irc-command-data-user](#irc-command-data-user)** -> verify -> **[irc-command-data-user](#irc-command-data-verified)** -> lưu [irc-database-central](#irc-database-central)

  * Thực thi: execute
    * client send ([action-direct](#action-direct)) đối với **[irc-command-data-user](#irc-command-data-user)**

    * client send ([action-direct](#action-direct)) hoặc ([action-command](#action-command)) đối với **[irc-command-data-user](#irc-command-data-verified)**

    * client wait event ([uirc-accepted](#uirc-accepted)) or ([uirc-rejected](#uirc-rejected)) để xác nhận chu trình

  * Giám sát: monitor
    * Theo dõi các command phát sinh khi user sử dụng remote control
    * client wait event ([uirc-executed](#uirc-executed))

## <a id='irc-database'></a>7. irc-database

* Yêu cầu:
  * Định danh được các command
  * irc-database-device : chỉ chứa các command **irc-command-data-verified**
  * irc-database-central: chứa 2 loại command data:
    * irc-command-data-user
    * irc-command-data-verified
  * irc-database-central: Nằm trên client và các thiết bị có khả năng lưu trữ - tìm kiếm - đồng bộ
  * irc-database-central: Chứa irc-database-device
  * irc-database-central: Quản lý **irc-database-device-multi-version** cho universal-irc-device để phục vụ cho việc thay đổi cấu trúc db trong tương lai

* Phân loại database

  * <a id='irc-database-central'></a>irc-database-central
    * db tập trung toàn bộ dữ liệu hiện có của mô hình

  * <a id='irc-database-device'></a>irc-database-device
    * db nằm trên universal-irc-device
    * Chứa 1 phần [irc-database-device](#irc-database-device)
    * tối ưu dung lượng lưu trữ < 1MB

* Kiến trúc irc-database-central
  > TODO

* Kiến trúc irc-database-device-version0
  * irc-database-device chia nhỏ thành page và block
    * irc-command-block: chứa 1 block data
    * irc-command-provider-page: chứa 1 vùng data của 1 nhà cung cấp thiết bị
  * page và block có kích thước cố định được định nghĩa trước

  * irc-database-device
    * irc-command-provider0-page0
      * irc-command-command-block0
      * irc-command-command-block1
      * ...
    * irc-command-provider0-page1
      * irc-command-command-block0
      * irc-command-command-block1
      * ...
    * irc-command-provider1-page0
      * irc-command-command-block0
      * irc-command-command-block1
      * ...
    * irc-command-provider1-page1
      * irc-command-command-block0
      * irc-command-command-block1
      * ...
    * ...

* Định danh irc-command
  * irc-command-block-id:
    * Để tối ưu dung lượng lưu trữ cho **irc-database-device**. Các command được chia block và định danh bẳng 3-bytes tối đa (2^24 == 16777216) irc-command-block
    * **irc-command-id == irc-command-block-id** (first block)
  * irc-command-block-size:
      * ước lượng: 32-bytes
    * command có độ dài <= irc-command-block-size -> irc-command-size = irc-command-block-size
    * command có độ dài > irc-command-block-size -> irc-command-size = n*irc-command-block-size với n = irc-command-block-size / irc-command-block-size

* irc-command-provider-page:
  * Vùng địa chỉ có size cố định được định nghĩa trước
  * 1 nhà cung cấp tùy số lượng thiết bị có thể có 1 hoặc nhiều page
  * irc-command-provider-pagesize:
      * ước lượng: (number-of-device * number-of-command-per-device * number-of-block-per-command) = 1024-blocks

* Đồng bộ irc-database-device
  > TODO

## <a id='mht-universal-irc-protocol-pubsub'></a>8. mht-universal-irc-protocol-pubsub: draff
  * Mục tiêu:
    * Xây dựng giao thức truyền nhận dựa trên giao thức pubsub đã implement trên mht-zigbee-gateway-js

  * Topic format
    * mqtt + iot-core
        > ${prefix}/action

        > ${prefix}/uirc-accepted

        > ${prefix}/uirc-rejected

        > ${prefix}/uirc-executed

    * socket.io
        > action

        > uirc-accepted
        
        > uirc-rejected

        > uirc-executed

  * payload
    * /action

      * Hướng:
        * irc-client > universal-irc-device

      * <a id='action-direct'></a>action-direct

        * chuyển tiếp data từ client > universal-irc-device
          * timestamp: Thời điểm phát sinh command (unix epoch time)
          * data: raw data encode dạng base64
          * hash: data sha1 hash encode base64
          * clientId:

        ```JSON
        {
          "type":"uirc-direct",
          "clientId": string,
          "data": base64,
          "hash": base64,
          "timestamp": number,
        }
        ```

      * <a id='action-command'></a>action-command
        * action-request-by-commandId
          * timestamp: Thời điểm phát sinh command (unix epoch time)
          * version: db version
          * commandId: irc-command-block-id
            * command data: được lấy từ irc-database-device theo commandId nếu khớp db version

        ```JSON
        {
          "type":"uirc-command",
          "version": number,
          "clientId": string,
          "commandId": number,
          "timestamp": number
        }
        ```

    * <a id='uirc-accepted'></a> shadow /uirc-accepted

      * Hướng:
        * universal-irc-device > irc-client

      * hash: data sha1 hash encode base64

      ```JSON
      {
        "type":"uirc-direct",
        "clientId": string,
        "hash": base64,
        "timestamp": number,
      }
      ```

      ```JSON
      {
        "type":"uirc-command",
        "version": number,
        "clientId": string,
        "commandId": number,
        "timestamp": number
      }
      ```
    
    * <a id='uirc-rejected'></a> shadow /uirc-rejected
      * Hướng:
        * universal-irc-device > irc-client

      * command reject reason:
        * unmatched-hash
        * invalid format
      ```JSON
      {
        "type":"uirc-direct",
        "invalidCode": number,
        "clientId": string,
        "hash": base64,
        "timestamp": number,
      }
      ```

      * command reject reason:
        * unmatched db version
        * invalid commandId
      ```JSON
      {
        "type":"uirc-command",
        "version": number,
        "clientId": string,
        "invalidCode": number,
        "commandId": number,
        "timestamp": number
      }
      ```

  * <a id='uirc-executed'></a> shadow /uirc-executed

    * Hướng:
      * universal-irc-device > irc-client

    * command nhận được từ uirc
    ```JSON
    {
      "data": base64,
      "hash": base64,
      "timestamp": number
    }
    ```