---
layout: post
title: Thay đổi logo khởi động trên một chiếc laptop Dell
description: Sau vài ngày vọc và tìm hiểu, mình đã thay đổi thành công UEFI Boot Logo của một chiếc laptop Dell. Cùng xem mình đã làm như nào nhé.
date: 2024-12-08 22:43:30 +0700
image: /images/result.jpg
---
Như các bạn đã biết (hoặc có thể không), mình là một đứa khá thích vọc vạch mấy thứ liên quan đến công nghệ 🥲 và bài viết này sẽ nói về một trong những trải nghiệm vọc vạch vừa qua của mình.

## Một tí background
Ông anh họ mình có nhờ mình cài lại Windows hộ trên một con máy và để lại ở nhà mình vài ngày. Con lap này cụ thể là một con Dell Latitude E7240, xài chip Intel Core i7-4600U, với 4GB RAM DDR3L (hơi ít để xài trong 2024 💀) và SSD mPCIe 128GB.

![image.jpg](/images/e7240_next_to_mbp.jpg)\
<sup>Dell Latitude E7240 (phải), để cạnh MacBook Pro 14-inch của mình (trái)</sup>

Thực sự, mình khá là thích con laptop này 😍. Con mắm này có 2 cổng RAM DDR3L, nhận max 1600MHz, kèm với **tận 3 slot mPCIe** cho khả năng mở rộng card Wi-Fi, card WAN, SSD mPCIe và có khi là cả GPU rời! Máy còn có thêm cả khay SIM (để xài khi máy có cắm WAN Card) và cả **detachable dual-storage BIOS module** (chip đôi BIOS có thể tháo rời) nữa. Máy cầm khá là nhẹ và mỏng, đồng thời các back cover được thiết kế để dễ dàng tháo rời, đúng là lap cho mấy doanh nhân có khác...!

![image.jpg](/images/detachable_dualstorage_bios.jpg)\
<sup>vờ lờ luôn</sup>

Vì con lap này xài khá là thích, nên mình cũng đã quyết định vọc vạch vài thứ trên đó luôn. Mình cũng đã thử undervolting để khiến cho con i7-4600U trong máy chạy đỡ nóng khi max xung nhịp Turbo, và hơn cả là đã mod thành công logo khởi động của máy. Như nào thì cùng follow mình nhé!

**Lưu ý:**
- Guide này chỉ áp dụng hoàn toàn cho Dell Latitude E7240 chạy phiên bản BIOS A08 (18/02/2014). Các máy khác có thể áp dụng được (khả năng cao nhất là cho các máy Dell xài AMI BIOS kèm checksum + Intel Boot Guard), nhưng mình sẽ không đảm bảo 100%.
- Nếu update BIOS thì sẽ phải làm lại. Muốn giữ ổn định thì tốt nhất không update, có vài hãng cũng push cả UEFI Capsule qua Windows Update luôn, nên chắc chắn rằng UEFI Capsule ở trong BIOS được tắt để tránh phải mệt...!
- Mình cũng không chắc 100% những thông tin ở bên dưới là chuẩn xác. Nếu có sai sót thì các bạn chỉ ra giúp mình nha!

## Mục tiêu
Tại vì đây không phải là con laptop của mình nên là mình sẽ cố gắng làm bản mod này mà không phải **tháo và program lại BIOS của máy bằng các phương pháp vật lí**.

Như các bạn thấy trên bức ảnh mình cầm cái module BIOS của máy, máy này sử dụng thiết kế dualstorage BIOS, có nghĩa là 2 chip sẽ chứa những phân vùng khác nhau của firmware máy. Việc tự dump hai chip này ra bằng kẹp nạp + CH341A sẽ khá tốn thời gian (không như những máy chỉ xài 1 chip BIOS), việc hợp nhất 2 image của chip này, mod và split lại cũng khá tốn thì giờ nữa, và đây cũng không phải là laptop mình (😭) nên mình đã lựa chọn việc **flash trực tiếp bằng các công cụ có sẵn trên Windows**.

## Nôm na là như này
Nói về mặt chung chung, quá trình mod sẽ khá đơn giản, có thể nói nôm na là như sau:

**Dump file firmware gốc máy ra -> Thay file logo của NSX bằng logo của mình -> Flash lại -> Profit!**

Tuy nhiên thực tế, quá trình nó lại không hoàn toàn đơn giản như vậy 😂 các bạn cứ đọc tiếp thì sẽ hiểu.

## Dump firmware gốc của máy (phần 1)
Cách đơn giản nhất và có lẽ là thẳng thắn nhất sẽ là extract trực tiếp từ chip qua 1 programmer như CH341A. Nhưng như đã nói ở trên, mình sẽ không làm thế.

Thì qua vài phút Google, mình đã tìm hiểu được rằng Intel ME có khả năng dump chip BIOS, miễn là firmware và phần cứng không có các restrictions do Intel và nhà sản xuất đặt ra bằng cách sử dụng **Intel Flash Programming Tool**. Flash Programming Tool sẽ đi theo suite (CS)ME System Tools của Intel, và mỗi một phiên bản của Intel ME sẽ xài một phiên bản System Tools khác nhau. Mình sẽ để link tool này ở [**đây**](https://mega.nz/folder/qdVAyDSB#FLCPaDVIsPYiy2TAUjD7RQ).

**Lưu ý:** Hãy chắc chắn rằng driver chipset và Intel ME của các bạn **đã được cài đặt cẩn thận**, nếu không những bước tới đây sẽ không thực hiện được.

Để biết được máy bạn đang xài phiên bản Intel ME gì, các bạn có thể tải [**Intel® Converged Security and Management Engine Version Detection Tool**](https://www.intel.com/content/www/us/en/download/19392/intel-converged-security-and-management-engine-version-detection-tool-intel-csmevdt.html) để xem.

![image.png](/images/csme_detection_result.png)

Như các bạn có thể thấy trong ảnh, mình đang xài ME version 9.5 nên sẽ tải ME System Tools 9.5. Sau khi các bạn tải về, giải nén, vào thư mục `Flash Programming Tool\WIN64` (hoặc `WIN32` nếu bạn xài Windows 32-bit, thay đổi `W64` thành `W32` trong guide nếu bạn đọc tiếp).

Tiếp tục, mở một cửa sổ command line **dưới quyền Administrator** và chạy lệnh `FPTW64.exe -bios -D bios.bin` để dump **phân vùng BIOS** của máy vào một file tên `bios.bin`.

![image.png](/images/bios_partition_dump_completed.png)

Như các bạn có thể thấy, mình đã dump được phân vùng BIOS của máy để tí nữa chỉnh sửa. Tiếp theo, mình thử viết lại chính cái dump vào BIOS xem là có viết được không.

![image.png](/images/write_fail_bios.png)

Như các bạn có thể thấy... mình không viết lại được, có vẻ như giờ, phân vùng BIOS đang trong trạng thái read-only. Thôi thì ít ra chúng ta cũng đã có backup lại được phân vùng BIOS... nhỉ?

Tuy nhiên, đây không phải là một bản backup BIOS hoàn chỉnh của máy, tại BIOS full nó bao gồm cả phân vùng Intel ME và vài cái lặt vặt nữa. Việc dump full content của BIOS khá quan trọng để phòng khi tí nữa máy mình kẻo có ăn c*t thì cũng còn có đường cứu...! Để dump full content của chip BIOS, mình sử dụng lệnh `FPTW64.exe -D fullbios.bin`.

![image.png](/images/fullbios_dump_error.png)

...tuy nhiên, mình cũng đã dính lỗi, lần này không dump được. Sau đó mình ngồi động não 3 phút và đã nghĩ ra được nguyên nhân. Nhập tiếp `FPTW64 -I` để hiện hết thông tin liên quan đến programmer để thấy rõ:

![image.png](/images/flash_info_image.png)

Lệnh dump sẽ dump toàn bộ chip nên yêu cầu toàn bộ phân vùng của BIOS phải có **read access**. Tuy nhiên như trên ảnh, các bạn có thể thấy rằng phân vùng Intel ME không có cả read và write access, khiến cho quá trình đó fail. Điều này chứng tỏ rằng đang có một cái khóa gì đó trong firmware hoặc hardware ràng buộc quá trình này.

<sup>(Nếu các bạn muốn tìm hiểu thêm về những kiểu "khóa" này, đọc phần [Notes](#notes) ở cuối bài.)</sup>

Vậy là giờ ta sẽ phải tự tay mod và flash mọi thứ mà phải động tay vào chip sao? Nhưng không, hi vọng giờ đây vẫn chưa hết! Thường thì các nhà sản xuất sẽ để một tùy chọn ẩn trong BIOS cho phép tắt những cái khóa liên quan đến CPU và firmware, nên tiếp theo mình sẽ tiến hành vọc vô cài đặt BIOS của máy, thử xem là có không.

## Vọc vào cài đặt BIOS của máy
Cái trang cài đặt BIOS của các bạn khi vào sẽ có một số lượng tùy chọn nhất định. Tuy nhiên, 99% rằng BIOS của bạn sẽ có các tùy chọn ẩn khác, do nhà sản xuất muốn giấu chúng để tránh người dùng táy máy 😂, và để biết được xem có những tùy chọn nào thì chúng ta cần phải extract được IFR từ trang Setup của BIOS.

Nói qua về IFR: IFR (**I**nternal **F**orm **R**epresentation) là 1 dạng binary dùng để chứa chữ, ảnh và một số file lặt vặt dùng để hiện lên 1 màn hình config trong 1 BIOS của một máy tính.

Để extract được IFR, chúng ta cần 2 tool: [**UEFITool**](https://github.com/LongSoft/UEFITool) và [**Universal-IFR-Extractor**](https://github.com/LongSoft/Universal-IFR-Extractor). Cái đầu sẽ dùng để extract IFR từ file BIOS của chúng ta và cái tiếp theo dùng để biến IFR về dạng file text đọc được.

Đầu tiên, mở file BIOS chúng ta vừa dump được trong UEFITool.

![image.png](/images/uefitool_first_bios_image.png)

Đập vào mắt chúng ta là hai thứ không mấy là vui cho lắm 😀, cụ thể là firmware của ta có Intel Boot Guard, nên khi mà thay đổi thứ gì trong firmware, **chúng ta phải cực kì cẩn thận!**

Bạn cứ hiểu rằng Intel Boot Guard nó như là UEFI Secure Boot nhưng là cho firmware của máy ấy. Một vài phần của firmware máy sẽ được kiểm tra lúc khởi động và tham chiếu với một key có sẵn trong **chipset** của máy. Key đó sẽ được nhà sản xuất **nhét vào trong chipset** từ lúc sản xuất, nên việc mod sai firmware sẽ khiến cho máy chúng ta **không thể khởi động**! Tuy nhiên, các nhà phát triển UEFITool đã giúp chúng ta khá nhiều ở đây. Những component chịu sự tra khảo của Intel Boot Guard đã được tô màu (như trên ảnh), nên chúng ta chỉ cần tránh những phần đó ra là được. (thank you so much 🙏)

Để tìm IFR của Setup trong BIOS chúng ta, invoke ô tìm kiếm bằng phím tắt Ctrl + F, rồi chúng ta tìm theo text theo tên một mục cài đặt mà đang hiện có trong cài đặt BIOS của chúng ta. Mình sẽ thử tìm `Secure Boot`.

![image.png](/images/ifr_find_uefitool.png)

Như các bạn thấy đấy, mình đã tìm được thành công được IFR của Setup BIOS. Tiếp theo, các bạn chuột phải, chọn `Extract as is` để lưu file ra dưới đuôi `.sct`.

Tuy nhiên, file này vẫn là dạng binary đặc biệt và ta sẽ không tự đọc được theo lẽ thông thường. Tiếp theo, ta cần sử dụng Universal-IFR-Extractor để chuyển đổi ra thành dạng text.

![image.png](/images/ifrextract.png)

```
ifrextract file.sct setup.txt
```
thay `file.sct` thành tên file đuôi `.sct` mà bạn vừa mới extract từ UEFITool.

Sau khi mở file này lên, ta thấy được tất cả các tùy chọn BIOS của chúng ta. Bạn sẽ cảm thấy rằng: "Sao nó lại nhiều thế!?", vì nó tính cả các option mà nhà sản xuất ẩn đi đó. Nhiều lắm phải không 😂⁉️

Sau khi mở file, mình dành vài phút lục qua các tùy chọn và tìm được một số thứ:

![image.png](/images/bios_lock_ifr.png)\
![image.png](/images/bios_interface_lock_ifr.png)\
![image.png](/images/me_fw_image_re_flash_ifr.png)\
![image.png](/images/var_store_name.png)\
<sup>Chú ý tới `VarOffset` và `VarStore` nhé, mình sẽ giải thích về sau!</sup>

Theo như mình tìm hiểu qua, `BIOS Lock` và `BIOS Interface Lock` là hai khóa trong firmware dùng để chặn flash BIOS. `Me Fw Image Re-Flash` dùng để cho phép Intel ME được đọc và được nạp lại, nói chung là cho phép read và write access luôn.

Nếu đúng như vậy thì tốt quá. **Nhưng có một vấn đề**. BIOS của chúng ta bị ẩn khá nhiều thứ và sẽ chắc chắn không có những tùy chọn này. Vậy chúng ta chỉnh kiểu gì? Thật may mắn cho chúng ta, mình lướt GitHub một hồi và tìm thấy câu trả lời: [**setup.var.efi**](https://github.com/datasone/setup_var.efi)! Đây là một ứng dụng được thiết kế để chạy trong UEFI Shell, giúp thay đổi các tùy chọn ẩn trong UEFI thông qua việc **điền offset**.

Đầu tiên, ta tải [**UEFI Shell**](https://github.com/pbatard/UEFI-Shell/releases) (`shellx64.efi`), copy vào một USB được format FAT32 theo đường dẫn `EFI\BOOT\BOOTX64.EFI`. Tiếp theo, ta tải binary của `setup_var.efi` về đường dẫn chính của USB. Boot máy vào USB bạn vừa tạo (nhớ tắt Secure Boot nhé!), ta sẽ nhận được màn hình sau:

![image.jpg](/images/uefi_shell.jpg)

Nếu ta đọc qua syntax của setup_var.efi trong cái README.md của nó, ta có thể thấy được cái syntax khá đơn giản. Để chỉnh `BIOS Lock` và `BIOS Interface Lock` thành `Disabled` và `Me Fw Image Re-Flash` thành `Enabled`, ta xài các lệnh sau:

```
setup_var.efi Setup:0x75=0x0
setup_var.efi Setup:0x77=0x0
setup_var.efi Setup:0x2BC=0x1
```

Giải thích sơ qua với ví dụ `BIOS Lock` nhé: Ta yêu cầu setup_var.efi vào store `Setup` (`VarStore: 0x2`), cho giá trị 0x0 (`Disabled`) vào offset `0x75` (`BIOS Lock`). Điều tương tự có thể được hiểu với 2 lệnh còn lại nha. Tiếp tục với...

## Dump firmware gốc của máy (phần 2)

Sau khi khởi động lại máy và chạy lại lệnh `FPTW64.exe -I`, ta thấy được rằng Intel ME đã có full read và write access 🙌

![image.png](/images/flash_info_image_P2.png)

Chúng ta cũng đã có thể viết đè lại vào phân vùng BIOS cái file `bios.bin` mà ta dump hồi nãy nữa 👀

![image.png](/images/write_bios_success.png)

Và hơn cả, ta đã có thể **dump full content chip BIOS ra** mà không gặp vấn đề gì!

![image.png](/images/fullbios_dump_success.png)

Với cái dump BIOS này, về sau nếu BIOS của các bạn có bị corrupt hay gì đi nữa thì các bạn cũng có thể sử dụng cái dump backup này để cứu máy về trạng thái ban đầu.

## Extract logo trong phân vùng BIOS
Cuối cùng mới đến phần thú vị các bạn ạ...! Tiếp tục mở file `bios.bin` trong UEFITool.

Thường trong BIOS, các nhà sản xuất sẽ sử dụng những định dạng ảnh như BMP, JPG và PNG. Việc của chúng ta là tìm chính xác cái boot logo của nhà sản xuất và thay thôi. Nhưng mà, khi nhìn qua đống này thì... nhìn **hơi loạn nhỉ!?**

![image.png](/images/uefitool_mess.png)\
<sup>mịe, nhìn thế này thì biết tìm đi đâu!?</sup>

Thực ra việc tìm ảnh nó không khó lắm, nếu các bạn động não đôi chút. Bằng cách sử dụng Hex pattern Search của UEFITool, các bạn có thể tìm ảnh trong vòng nháy mắt.

![image.png](/images/find_bmp_hex.png)

Như ảnh trên là mình đang sử dụng mục Search, để là `Body only`, và đang tìm hex `42 4D` - **là mã hex khởi đầu của mọi ảnh bitmap**. Trên thanh output search, UEFITool đã tìm được rất nhiều mục, nhưng bạn chỉ nên chú ý tới những mục hiện `Raw section`, vì nó không phải là một binary hay image liên quan đến UEFI mà tool nhận dạng được, mà là 1 file hoàn toàn khác.

![image.png](/images/found_boot_logo.png)

Và sau đó, mình thử ấn `Extract body` của cái mục đó, lưu file dưới đuôi `.bmp` thử và voila! Mình đã tìm được thành công boot logo của firmware. Cho bạn nào muốn biết thì con Dell này của mình có boot logo ở trong mục có GUID là `EB3001D5-F827-4938-95E2-5C8F644B966F`, search bằng GUID là thấy.

Các bạn có thể làm điều tương tự với các định dạng ảnh khác, ví dụ như JPG với header `FF D8 FF`, hoặc PNG với header `89 50 4E 47 0D 0A 1A 0A`.

## Chỉnh sửa ảnh đã extract và thay thế với ảnh cũ
Khi extract ảnh và xem qua, ta thấy được tính trạng của file như sau: **1280x1280 với 4-bit màu**.

![image.png](/images/image_properties.png)

Khi chúng ta edit, chúng ta cần chắc chắn rằng thuộc tính của ảnh mới về kích cỡ và bit màu sẽ **giống với ảnh cũ**. Đặc biệt với những format như JPG và PNG, ảnh mới cố gắng không được quá nặng so với ảnh cũ để BIOS còn chỗ chứa lưu.

Cách edit rất đơn giản thôi, mở phần mềm chỉnh ảnh bạn thích, sửa ảnh và khi lưu ảnh, kích cỡ và bit màu phải giống ảnh cũ.

![image.png](/images/new_logo_show.png)

Sau khi đã sửa xong ảnh, bạn cần [**UEFITool 0.28.0**](https://github.com/LongSoft/UEFITool/releases/tag/0.28.0) để có thể chỉnh sửa file BIOS. Lí do là những bản UEFITool xài engine mới (UEFITool NE) chưa hỗ trợ image editing.

Mở UEFITool 0.28.0 lên, tìm lại entry chứa file BIOS như cách đã tìm khi ở UEFITool bản mới. Khi đã tìm thấy, ấn chuột phải rồi `Replace body` và chọn file ảnh bạn đã sửa. Khi nào nó hiện `Remove` với `Replace` là oke.

![image.png](/images/replace_body_context.png)
![image.png](/images/rebuild_remove.png)

Cơ mà đợi đã... **CHỚ LƯU IMAGE VÀ THOÁT!** Còn nhớ lúc đầu mình bảo rằng, máy của chúng ta **có Intel Boot Guard** không? Để mình giải thích:

![image.png](/images/uefitool_comparison.png)

Đây là hai cửa sổ UEFITool, bên trái là file firmware của chúng ta trong UEFITool 0.28.0 sau khi mod, bên phải là UEFITool New Engine. Nếu các bạn còn nhớ, thì những phần tô màu ở bên UEFITool NE **chỉ những phần được Intel Boot Guard** bảo vệ.

Khi UEFITool 0.28.0 modify một volume, bằng một cách ma quỷ nào đó, đôi lúc nó cũng sẽ tình cờ modify một volume khác luôn. Những volume nào bị modify thì các bạn sẽ thấy hiện `Rebuild` ở cột Action.

![image.png](/images/rebuild_indicator_uefitool.png)

Và hay chưa...! UEFITool 0.28.0 đang có ý định modify volume cuối cùng trong cái UEFI image, nếu bạn tham chiếu qua bên cửa sổ UEFITool NE bên cạnh thì các bạn cũng có thể thấy rằng **volume đó đang được bảo vệ bởi Boot Guard (màu đỏ ở dưới cùng)!** Có nghĩa là nếu các bạn save file ở trạng thái này và đem flash vào máy luôn, **máy của các bạn sẽ brick NGAY LẬP TỨC.**

![image.png](/images/bootguard_comparison.png)

Để có thể mod thành công, các bạn ấn chuột phải vào những phần nào liên quan đến Boot Guard ở bên UEFITool 0.28.0 và ấn `Do not rebuild`, lúc này dòng chữ sẽ chuyển từ `Rebuild` thành `Do not rebuild`, và lúc lưu thì UEFITool 0.28.0 sẽ không đả động gì vào phần đó nữa.

![image.png](/images/do_not_rebuild.png)

## Flash và... tận hưởng!
Sau khi kiểm tra mọi thứ xong xuôi, các bạn save file lại dưới 1 file tên khác, như trường hợp của mình sẽ để là `newbios.bin`, và ném qua flash bằng `FPTW64.exe -bios -F newbios.bin` thôi!

![image.png](/images/modded_flash_success.png)

Sau khi flash xong, các bạn **tắt hẳn máy** (đừng reboot) và bật lại lên để chip BIOS mới sử dụng lại content mới mà vừa được flash vào. Và... logo boot mới say hi!

![image.jpg](/images/result.jpg)

## Notes
- Blog này được làm để miêu tả quá trình mod trên một chiếc Dell Latitude E7240, nhưng mình cũng đã mod thành công 1 chiếc Dell Vostro 15 3568 qua call online với thằng bạn [Nico](https://www.facebook.com/silly1nico). [Proof ở đây 🤓](/images/proof_nico.jpg)
- Để tìm hiểu thêm về các loại protection của chip SPI trong máy, bạn có thể check qua [**bài blog này của eclypsium**](https://eclypsium.com/blog/firmware-security-realizations-part-3-spi-write-protections/) hoặc [**thread này trên Win-Raid**](https://winraid.level1techs.com/t/guide-unlock-intel-flash-descriptor-read-write-access-permissions-for-spi-servicing/32449).
- Nếu bạn muốn, bạn cũng có thể dump và flash BIOS qua Afuwin hoặc các cách khác nếu thích, dù mình chưa thử (và mình cũng có cảm giác là cái Intel ME nó an toàn hơn 🥲)
- Post này nếu được áp dụng linh hoạt thì có thể áp dụng trên các máy model khác, thậm chí là hãng khác luôn. Đặc biệt là lũ PC, tại consumer PC thì hiếm có mainboard trang bị Intel Boot Guard, trừ những dòng high-end hoặc vài series pre-built của mấy hãng lớn.

**Alright byee, hẹn gặp lại các bạn ở một post khác ⭐️**

