    # Phân Tích Ảnh Số: Ngưỡng Hóa – Biên – Góc – Matching Đặc Trưng

    **Sinh viên thực hiện:** Lưu Võ Phương Mai <br>
    **MSSV:** 2374802010299 <br>
    **Môn học:** Nhập môn Xử lý ảnh số <br>
    **Giảng viên:** Đỗ Hữu Quân <br>


    ## Giới thiệu

    Bài lab này tập trung vào chuỗi thao tác cơ bản đến nâng cao trong xử lý ảnh số:

    * Ngưỡng hóa ảnh và phân vùng đối tượng
    * Phát hiện biên bằng toán tử gradient
    * Phát hiện điểm góc (corner) bằng Harris
    * So khớp đặc trưng giữa hai ảnh thực tế

    Các kỹ thuật này đặt nền tảng cho các bài toán thị giác máy tính cao cấp như: phân vùng ảnh, nhận diện vật thể, ghép ảnh (image registration), theo dõi chuyển động,...

    ---

    ## Công nghệ sử dụng

    * **Python:** ngôn ngữ chính
    * **Pillow (PIL):** đọc/ghi ảnh
    * **NumPy:** thao tác ma trận ảnh
    * **ImageIO:** hỗ trợ định dạng ảnh hiện đại
    * **OpenCV (cv2):** hỗ trợ xử lý nâng cao (Sobel, màu sắc)
    * **Matplotlib:** hiển thị ảnh trực quan
    * **SciPy.ndimage:** tính đạo hàm, làm mịn
    * **Scikit-image (skimage):** hàm xử lý ảnh cao cấp: `label`, `regionprops`, `corner_harris`, `corner_peaks`

    ---

    ## Chi tiết các kỹ thuật đã triển khai


    ### 1. Ngưỡng hóa ảnh bằng phương pháp Otsu + Labeling + Bounding Box

    **Mục tiêu:** phân đoạn ảnh nhị phân, phát hiện đối tượng

    #### Quy trình:

    * Chuyển ảnh grayscale → dùng ngưỡng Otsu để tạo ảnh nhị phân
    * Dùng `label()` để đánh nhãn các vùng
    * Áp dụng `regionprops()` để lấy thông tin: vùng, centroid, bounding box
    * Vẽ các hộp bao quanh đối tượng

    #### Thư viện liên quan:

    `threshold_otsu`, `label`, `regionprops`, `matplotlib.patches`

    ---

    ### 2. Phát hiện biên đơn giản bằng dịch ảnh

    **Ý tưởng:** Biên được làm nổi bật khi lấy hiệu độ sáng giữa ảnh gốc và ảnh bị dịch.

    #### Cách thực hiện:

    * Dịch ảnh grayscale sang phải 1 pixel (hàm `nd.shift`)
    * Tính sai khác tuyệt đối giữa hai ảnh → kết quả là biên

    #### Ưu điểm:

    * Đơn giản, không dùng đạo hàm
    * Là cách trực quan để hiểu "biên" là nơi có sự thay đổi nhanh cường độ

    ---

    ### 3. Phát hiện biên bằng đạo hàm Sobel

    **Ý tưởng:** Tính gradient ảnh theo trục x và y để tìm đường biên.

    #### Công thức:

    * $G_x = \frac{dI}{dx}$, $G_y = \frac{dI}{dy}$
    * $G = |G_x| + |G_y|$

    #### Cách làm:

    * Dùng `nd.sobel` theo 2 hướng
    * Cộng lại giá trị tuyệt đối → biên tổng hợp

    #### Kết quả:

    Ảnh biểu diễn biên rõ ràng hơn so với dịch ảnh.

    ---

    ### 4. Phát hiện góc Harris (tự cài đặt)

    **Ý tưởng:** Góc là nơi gradient thay đổi lớn theo cả hai hướng.

    #### Công thức:

    * Ma trận C cấu trúc cục bộ:
    C = [ [Ix^2, Ix·Iy],
        [Ix·Iy, Iy^2] ]
    
    * Đáp ứng góc Harris:

    R=det(C)−α(trace(C))^2

    #### Triển khai:

    * Tính đạo hàm bằng Sobel
    * Làm mịn với Gaussian
    * Tính R theo công thức trên
    * Vẽ bản đồ điểm góc

    ---

    ### 5. Biến đổi Hough phát hiện đường thẳng (tự cài đặt)

    **Ý tưởng:** Ánh xạ điểm ảnh sáng lên không gian (ρ, θ) để tìm đường thẳng

    #### Các bước:

    * Với mỗi điểm sáng: tính ρ theo θ từ 0–90°
    * Tích lũy vào Hough space
    * Trả về ảnh thể hiện các đường thẳng mạnh

    #### Ưu điểm:

    * Dễ hiểu, tự cài đặt không cần OpenCV
    * Có thể áp dụng cho ảnh edge đã lọc sẵn

    ---

    ### 6. Harris Corner Detection bằng thư viện `skimage`

    **Mục tiêu:** Phát hiện góc tự động từ ảnh thật (`bird.png`)

    #### Cách làm:

    * Chuyển ảnh sang grayscale
    * Áp dụng `corner_harris()` với hệ số k = 0.001
    * Hiển thị ảnh kết quả phản hồi điểm góc

    #### Khác biệt:

    * Không phải tự tính tay như đoạn 4
    * Nhanh, gọn hơn — nhưng mất chi tiết kiểm soát

    ---

    ### 7. So khớp đặc trưng giữa hai ảnh thực tế

    **Mục tiêu:** Dò tìm các điểm tương đồng giữa ảnh `bird.png` và `dalat.jpg`

    #### Quy trình:

    1. Dò điểm góc bằng `corner_harris`
    2. Lọc lại các góc tốt bằng `corner_peaks`
    3. Cắt patch 11×11 quanh từng điểm góc → tạo vector đặc trưng
    4. Tính khoảng cách Euclidean giữa các vector
    5. Ghép các cặp đặc trưng có khoảng cách nhỏ nhất
    6. Hiển thị ảnh ghép với đường nối cặp điểm khớp

    #### Kết quả:

    Ảnh hiển thị các mối liên kết giữa 2 ảnh dựa trên matching descriptor.
    Rất phù hợp để làm nền tảng cho bài toán nhận dạng vật thể & ghép ảnh.

    ---

    ## Cấu trúc file

    ```
    ├── bai_lab.ipynb / .py     # File code xử lý chính
    ├── geometric.png           # Ảnh đơn hình học
    ├── bird.png                # Ảnh thật 1
    ├── exercise/dalat.jpg      # Ảnh thật 2
    ├── label_output.png        # Kết quả labeling
    ├── README.md               # Tài liệu hướng dẫn (file này)
    ```
    ---

    ## Hướng dẫn 

    ### 1. Cài đặt thư viện cần 

    ```bash
    pip install opencv-python
    ```

    ### 2. Chạy file chính

    * Mở Jupyter Notebook trên VSCode/Colab
    * Chạy từng cell để xem kết quả của các thuật toán
    * Đảm bảo các file ảnh được đặt đúng thư mục

    ### 3. Tuỳ chỉnh

    * Thử với ảnh khác: đổi `bird.png`, `dalat.jpg`
    * Điều chỉnh `k`, `gamma`, `threshold`, v.v. để quan sát sự thay đổi
    * Có thể kết hợp thêm edge detection, lọc nhiễu, v.v. để nâng cao chất lượng đầu vào


