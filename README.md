# Xem-t-ng-s-ti-n-mua-tr-n-shopee-m-i
THỐNG KÊ SỐ TIỀN ĐÃ MUA Ở SHOPEE CODE MỚI
CÔNG CỤ THÔNG KÊ TỔNG ĐƠN HÀNG VÀ SỐ TIỀN ĐÃ MUA TRÊN SHOPEE

📈 Công cụ thống kê chi tiêu trên Shopee
Shopee đã thay đổi API nên các mã code hiện tại sẽ không chạy được nữa. Dựa trên mã code của người trước mình up lên code đã có thay đổi phù hợp với chế độ của Shopee hiện tại, mong các bạn thành công

✨Tính năng

. Thống kê tổng số tiền đã chi tiêu

. Đếm tổng số đơn hàng đã hoàn thành

. Đếm tổng số sản phẩm đã mua

. Tính tổng tiền tiết kiệm từ mã giảm giá

. Đánh giá mức độ "nghiện" Shopee của bạn


Dưới đây là cách để xem tổng số tiền đã mua hàng trên Shopee trên máy tính

BƯỚC 1: Bạn click vào link "https://shopee.vn/user/purchase/"

BƯỚC 2: Bấm F12 -> tiếp tục chọn "Sources"

<img width="1163" height="226" alt="image" src="https://github.com/user-attachments/assets/844f21d5-f75d-4081-bad7-26a0c69d915a" />

BƯỚC 3: Tiếp tục làm theo thứ tự trong ảnh

<img width="1448" height="396" alt="image" src="https://github.com/user-attachments/assets/7bfe9205-1117-4e54-a0ea-f00c6749b8f7" />

BƯỚC 4: Tiếp tục chọn new Snippets -> bạn coppy toàn bộ code bên dưới vào (có thể bấm ctrl S để lần sau chạy tiếp)

<img width="967" height="627" alt="image" src="https://github.com/user-attachments/assets/099a20f7-856c-41d7-b2c2-59e3c045bf40" />

BƯỚC 5: Bấm ctrl + enter (sau đó chờ để code xử lý)

-------------------------------------------------------------
var tongDonHang = 0;
var tongTienTietKiem = 0;
var tongtienhang = 0;
var tongtienhangchuagiam = 0;
var tongSanPhamDaMua = 0;
var offset = 0;
var limit = 20;

function xemBaoCaoThongKe() {
    var xhttp = new XMLHttpRequest();
    xhttp.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            try {
                var response = JSON.parse(this.responseText);
                console.log('Response:', response); // Debug
                
                // Kiểm tra cấu trúc dữ liệu mới của Shopee
                var orders = response?.data?.order_data?.details_list || 
                            response?.data?.details_list || 
                            response?.data?.order_list || [];
                
                if (!orders || orders.length === 0) {
                    console.log('Không còn đơn hàng nào hoặc API đã thay đổi cấu trúc');
                    hienThiKetQua();
                    return;
                }
                
                tongDonHang += orders.length;
                var conTiep = orders.length >= limit;
                
                orders.forEach(order => {
                    try {
                        // Lấy tổng tiền đơn hàng (đã giảm giá)
                        var finalTotal = order?.info_card?.final_total || 
                                       order?.final_total || 
                                       order?.total_price || 0;
                        tongtienhang += finalTotal / 100000;
                        
                        // Lấy thông tin sản phẩm
                        var orderCards = order?.info_card?.order_list_cards || 
                                       order?.order_list_cards || [];
                        
                        orderCards.forEach(card => {
                            var productInfo = card?.product_info || card;
                            var itemGroups = productInfo?.item_groups || 
                                           [{ items: productInfo?.items || [] }];
                            
                            itemGroups.forEach(group => {
                                var items = group?.items || [];
                                items.forEach(item => {
                                    var orderPrice = item?.order_price || 
                                                   item?.price || 0;
                                    var amount = item?.amount || 
                                               item?.quantity || 0;
                                    
                                    tongSanPhamDaMua += amount;
                                    tongtienhangchuagiam += (orderPrice / 100000) * amount;
                                });
                            });
                        });
                    } catch (err) {
                        console.error('Lỗi xử lý đơn hàng:', err);
                    }
                });
                
                offset += limit;
                
                if (conTiep) {
                    console.log('Đã thống kê: ' + tongDonHang + ' đơn hàng. Đang lấy thêm...');
                    setTimeout(xemBaoCaoThongKe, 500); // Delay để tránh spam API
                } else {
                    hienThiKetQua();
                }
                
            } catch (err) {
                console.error('Lỗi parse JSON:', err);
                console.log('Response text:', this.responseText);
                hienThiKetQua();
            }
        } else if (this.readyState == 4) {
            console.error('Lỗi API, status:', this.status);
            hienThiKetQua();
        }
    };
    
    xhttp.open("GET", "https://shopee.vn/api/v4/order/get_order_list?list_type=3&offset=" + offset + "&limit=" + limit, true);
    xhttp.send();
}

function hienThiKetQua() {
    tongTienTietKiem = tongtienhangchuagiam - tongtienhang;
    
    console.log("================================");
    console.log("%c" + getPXGCert(tongtienhang), "font-size:26px; font-weight:bold;");
    console.log("================================");
    console.log("%cSố tiền ĐÃ CHI vào Shopee: %c" + formatPrice(tongtienhang) + " vnđ", 
        "font-size: 20px;", "font-size: 26px; color:orange; font-weight:700");
    console.log("================================");
    console.log("%cTổng đơn hàng: %c" + formatPrice(tongDonHang) + " đơn", 
        "font-size: 20px;", "font-size: 20px; color:green");
    console.log("%cSố sản phẩm: %c" + formatPrice(tongSanPhamDaMua) + " sản phẩm", 
        "font-size: 20px;", "font-size: 20px; color:#fc0000");
    console.log("%cTiền TIẾT KIỆM nhờ giảm giá: %c" + formatPrice(tongTienTietKiem) + " vnđ", 
        "font-size: 18px;", "font-size: 18px; color:green");
    console.log("%c💰 TỔNG TIẾT KIỆM: %c" + formatPrice(tongTienTietKiem) + " vnđ", 
        "font-size: 24px;", "font-size: 24px; color:orange; font-weight:700");
    console.log("================================");
}

function getPXGCert(pri) {
    if (pri <= 10000000) {
        return "HÊN QUÁ! BẠN CHƯA BỊ SHOPEE GÂY NGHIỆN 😍";
    } else if (pri <= 50000000) {
        return "THÔI XONG! BẠN BẮT ĐẦU NGHIỆN SHOPEE RỒI 😂";
    } else if (pri < 80000000) {
        return "ỐI GIỜI ƠI! BẠN LÀ CON NGHIỆN SHOPEE CHÍNH HIỆU 😱";
    } else {
        return "XÓA APP SHOPEE THÔI! BẠN NGHIỆN SHOPEE NẶNG QUÁ RỒI 😝";
    }
}

function formatPrice(number, fixed = 0) {
    if (isNaN(number)) return '0';
    number = number.toFixed(fixed);
    return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
}

// Bắt đầu
console.log('Bắt đầu thống kê... Vui lòng đợi!');
xemBaoCaoThongKe();
