---
layout: post
title: Bypass khảo sát trên web của trường đại học
description: Mình đã bypass thành công khảo sát trên web của đại học mình đang theo học như nào?
date: 2024-11-26 11:34:30 +0700
image: /images/survey-required.png
---
Như các bạn đã biết (hoặc không?), mình đang theo học tại một trường đại học liên quan đến công nghệ khá là mới ở Việt Nam. Mình sẽ tạm thời không nhắc tới tên ở trong post này vì một vài lí do, nếu bạn muốn biết mình học trường gì thì ghé qua trang chủ (portfolio) của mình thì sẽ biết: [**https://qtpc.tech**](https://qtpc.tech) 😂

## Vấn đề nó là như này...
Trường mình nó có cái web, khá là "xịn", nhưng có cái cơ chế mà mình không thấy mặn mà lắm, nó như này:

![image.png](/images/survey-required.png)

Mỗi một lần vào web, hệ thống sẽ bắt sinh viên thực hiện tất cả các khảo sát hiện tại của trường, học sinh phải thực hiện hết thì UI chính của web mới hiện cho sinh viên thao tác.

Thực sự mình thấy điều này **khá** là bất tiện, tại cái web thực hiện khảo sát của trường thực sự lỏ 😂, một lỗi ví dụ là khi mình đã hoàn thành xong một khảo sát và apply, nhưng hệ thống cứ khăng khăng thông báo rằng mình chưa hoàn thành khảo sát (yes, mình đã điền vào tất cả các ô chữ và chọn tất cả các ô tròn, kể cả các ô không bắt buộc)

![image.png](/images/survey-not-completed.png)

Mình là một người khá là chỉn chu, thật thà và cẩn thận (có thể là real hoặc 🧢, tùy bạn đánh giá 🥲) trong những lúc thực hiện những đợt khảo sát như này, nên mình thường sẽ dành thời gian rảnh (khi bản thân thực sự bình tĩnh) để làm. Còn những lúc mình cần vào web trường để làm những việc cần thiết khác (check lịch học, gửi form, check tiền học, etc), thì mình **thực sự không muốn thực hiện điền khảo sát**.

Nhiều bạn sẽ nghĩ rằng: "Ủa vậy bạn sẽ chạy cái script này mãi trên web trường để không phải điền khảo sát hả? 😩". Không không, như đã nói ở trên, mình muốn thực sự làm khảo sát vào những lúc rảnh và thực sự bình tĩnh nhất, khi không có việc gấp hay deadline ập đến. Mình sẽ chỉ kích hoạt cái script này khi không muốn thực hiện khảo sát và sẽ tắt script và thực hiện vào lúc khác.

Vậy nên...

## Tiến hành viết userscript bypass thông báo khảo sát
Mình không chắc là những thứ mình viết dưới đây có 100% đúng hay không, tại mình không phải dân chuyên 😭 có gì góp ý sửa đổi nếu được nha

Behavior mặc định của trang như mình thấy bằng mắt được như sau:
- Khi vào web, web sẽ check xem sinh viên có các đợt khảo sát nào chưa thực hành không
- Nếu có, show một popup native alert báo "Bạn cần hoàn thành các phiếu khảo sát sau." với một nút OK.
- Trang web sẽ đợi vài giây và redirect ra trang thực hiện khảo sát, trên link có token (kiểu dạng như session ID !? mình không có rõ lắm) và ID người thực hiện khảo sát.

![image.png](/images/url_survey.png)

Tiếp theo, mình tiến hành inspect website và tìm được những script js được load khi bắt đầu load, mục đích là để tìm xem đâu là đối tượng đã load cái alert box:

![image.png](/images/inspect_uni_web.png)

Mình tìm được một file có vẻ là đối tượng hiện alert box cho người dùng mang tên `systemroot.js`

![image.png](/images/alert_box_code.png)

Vậy là mình đã nắm được rằng, function `ChuyenChucNang()` nắm vai trò hiện alert box và redirect sinh viên tới web để thực hiện khảo sát.

Giải thích đơn giản, function này gửi một HTTP POST request đến API. POST request này bao gồm action `CMS_TOKEN/CreateTicket`, ID của user thực hiện, ID của app và ticket ID (lấy bằng `edu.util.uuid()`). Khi POST request thành công (`data.Success`), và trong request đó có id khảo sát (`data.Id`) thì tiến hành hiện alert box và điều hướng người dùng đến link khảo sát.

Thực ra lúc đầu, mình không đọc kĩ code nên mình đã nghĩ là chỉ cần override được alert box là skip được cả quá trình 😂 code như sau:
```js
// Override the native alert function
window.alert = function() {
    // Prevent the alert from showing
    console.log('Alert blocked!');
};
```

Đúng là nó có block được alert thật, nhưng về sau thì web vẫn redirect mình về link khảo sát. Hihi, mình nặn óc ra vài phút thì nghĩ ra được một giải pháp ổn áp nhất: **Override hẳn luôn cái function `ChuyenChucNang()` này**, tại mình thấy cái function này không làm cái gì quá quan trọng ngoài query trạng thái thực hiện khảo sát của người dùng.

Cuối cùng, đây là sản phẩm sau cùng của mình:

```js
(function() {
    'use strict'; // chế độ nghiêm ngặt

    // Đợi script load
    const waitForScript = setInterval(() => {
        // Check đối tượng edu xem đã được load chưa, check xem ChuyenChucNang() có phải là một hàm tồn tại hay không
        if (typeof edu !== "undefined" && edu.system && typeof edu.system.ChuyenChucNang === "function") {
            // Thực hiện ghi đè khi đã tìm thấy đối tượng
            clearInterval(waitForScript);

            // Lưu hàm gốc vào originalChuyenChucNang, phòng khi sau nếu có muốn dùng lại trong web console
            const originalChuyenChucNang = edu.system.ChuyenChucNang;

            // Ghi đè hàm ChuyenChucNang() gốc của web bằng phiên bản của userscript này
            edu.system.ChuyenChucNang = function() {
                // nếu hàm ChuyenChucNang() được gọi sau khi ghi đè, thay vì đưa người dùng đến trang khảo sát thì nó sẽ in ra console như sau
                console.log("ChuyenChucNang was called, but execution is blocked.");
            };

            // thông báo trong console rằng hàm ChuyenChucNang() đã được ghi đè thành công
            console.log("Userscript: ChuyenChucNang method has been successfully overridden.");
        }
    }, 100); // Check mỗi 100ms (1/10 giây)

    // Giới hạn thời gian kiểm tra hàm lúc đầu còn 10 giây.
    setTimeout(() => {
        clearInterval(waitForScript);
        console.log("Userscript: Stopped checking for ChuyenChucNang after timeout.");
    }, 10000);
})();
```

Các bạn có thể sử dụng cái userscript này bằng cách tải từ link [**https://stash.qtpc.tech/skipchuyenchucnang.user.js**](https://stash.qtpc.tech/skipchuyenchucnang.user.js), và sử dụng với trình đọc userscript các bạn muốn (ví dụ như [**Tampermonkey**](https://www.tampermonkey.net), hoặc nếu bạn xài Safari thì [**Userscripts**](https://github.com/quoid/userscripts)).

**LƯU Ý:** Copy cái code snippet trên và paste không vào sẽ không chạy được, tại nó thiếu những metadata cần thiết để cho trình userscript có thể hoạt động.

**Alright byee, hẹn gặp lại các bạn ở một post khác ⭐️**