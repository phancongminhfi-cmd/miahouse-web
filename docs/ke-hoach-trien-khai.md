# Kế hoạch triển khai miahouse-web

Tài liệu này cụ thể hóa đặc tả trong `README.md` thành kế hoạch kỹ thuật. Trọng tâm là **Giai đoạn 1: Website giới thiệu và nhận khách**, đồng thời thiết kế dữ liệu và phân quyền sẵn để mở rộng sang giai đoạn 2 đến 4 mà không phải làm lại.

## 1. Phạm vi phiên bản đầu tiên (MVP)

Có trong MVP:

- Trang công khai: trang chủ, danh sách phòng, chi tiết phòng, trang tòa nhà, giới thiệu, hướng dẫn thuê, liên hệ, landing page theo khu vực.
- Form đăng ký tư vấn và đặt lịch xem phòng, lưu vào cơ sở dữ liệu và gửi email báo cho nhân viên.
- Nút gọi điện, Zalo (và WhatsApp cho khách quốc tế) cố định trên điện thoại.
- Trang quản trị: quản lý tòa nhà, phòng, tiện nghi, hình ảnh, yêu cầu tư vấn (danh sách và cập nhật trạng thái), banner, câu hỏi thường gặp, thông tin liên hệ.
- Giao diện tiếng Việt hoàn chỉnh, cấu trúc đa ngôn ngữ sẵn cho tiếng Anh.
- SEO kỹ thuật: URL riêng, metadata, sitemap, dữ liệu có cấu trúc (JSON-LD), Google Analytics 4.

Để sau MVP:

- CRM đầy đủ, phân công nhân viên, lịch xem phòng dạng lịch (giai đoạn 2).
- Giữ chỗ trực tuyến, thanh toán cọc, hợp đồng (giai đoạn 3).
- Quản lý người thuê, công nợ, điện nước, bảo trì (giai đoạn 4).
- Bài viết và tin tức: bảng `articles` tạo sẵn, giao diện quản trị bài viết làm sau nếu chưa cần ngay.

## 2. Kiến trúc và công nghệ

| Thành phần | Lựa chọn | Lý do |
| --- | --- | --- |
| Framework | Next.js (App Router) + React + TypeScript | Render phía server tốt cho SEO, một mã nguồn cho cả website và trang quản trị. |
| Giao diện | Tailwind CSS + shadcn/ui | Nhanh, nhẹ, dễ tùy biến theo màu nhận diện. |
| Đa ngôn ngữ | `next-intl`, URL dạng `/vi/...` và `/en/...` | Mỗi ngôn ngữ có URL riêng, tốt cho SEO. |
| Cơ sở dữ liệu | Supabase (PostgreSQL) | Có sẵn xác thực, lưu trữ ảnh và phân quyền theo dòng (RLS). |
| Xác thực quản trị | Supabase Auth (email + mật khẩu hoặc magic link) | Chỉ nhân sự nội bộ đăng nhập; khách không cần tài khoản. |
| Lưu ảnh | Supabase Storage + tối ưu ảnh của Next.js | Ảnh tự nén và đổi kích thước theo thiết bị. |
| Email thông báo | Resend (hoặc SMTP của công ty) | Gửi email khi có yêu cầu mới. |
| Bản đồ | Google Maps Embed / Maps JavaScript API | Hiển thị vị trí tòa nhà và khu vực. |
| Chống spam form | Cloudflare Turnstile + giới hạn số lần gửi | Không làm phiền khách như CAPTCHA truyền thống. |
| Triển khai | Vercel (website) + Supabase (dữ liệu) | Mỗi PR có bản xem thử riêng để duyệt trước khi lên chính thức. |
| Phân tích | Google Analytics 4 + Google Search Console | Đo lượt xem, chuyển đổi và nguồn khách. |

## 3. Cấu trúc URL

Trang công khai (ví dụ cho tiếng Việt; tiếng Anh dùng tiền tố `/en` với slug tiếng Anh):

| Trang | URL |
| --- | --- |
| Trang chủ | `/vi` |
| Danh sách phòng | `/vi/phong` (bộ lọc nằm trên query, ví dụ `?khu-vuc=quan-1&gia-max=10000000`) |
| Chi tiết phòng | `/vi/phong/[slug-phong]` |
| Danh sách tòa nhà | `/vi/toa-nha` |
| Trang tòa nhà | `/vi/toa-nha/[slug-toa-nha]` |
| Landing page khu vực | `/vi/khu-vuc/[slug-khu-vuc]`, ví dụ `/vi/khu-vuc/quan-1` |
| Landing page khu vực theo loại phòng | `/vi/khu-vuc/[slug-khu-vuc]/[loai-phong]`, ví dụ `/vi/khu-vuc/quan-1/can-ho-dich-vu` |
| Giới thiệu | `/vi/gioi-thieu` |
| Hướng dẫn thuê | `/vi/huong-dan-thue` |
| Liên hệ | `/vi/lien-he` |

Trang quản trị (không đánh chỉ mục tìm kiếm):

| Trang | URL |
| --- | --- |
| Đăng nhập | `/admin/login` |
| Tổng quan | `/admin` |
| Tòa nhà | `/admin/properties` |
| Phòng và căn hộ | `/admin/units` |
| Tiện nghi | `/admin/amenities` |
| Yêu cầu tư vấn | `/admin/leads` |
| Nội dung (banner, FAQ, liên hệ) | `/admin/content` |
| Người dùng | `/admin/users` |

## 4. Cấu trúc thư mục dự kiến

```text
miahouse-web/
├── app/
│   ├── [locale]/            # Trang công khai theo ngôn ngữ
│   │   ├── page.tsx         # Trang chủ
│   │   ├── phong/
│   │   ├── toa-nha/
│   │   ├── khu-vuc/
│   │   ├── gioi-thieu/
│   │   ├── huong-dan-thue/
│   │   └── lien-he/
│   ├── admin/               # Trang quản trị
│   ├── api/                 # Endpoint cần thiết (webhook, revalidate)
│   ├── sitemap.ts
│   └── robots.ts
├── components/
│   ├── ui/                  # Thành phần giao diện dùng chung
│   ├── public/              # Thành phần trang công khai
│   └── admin/               # Thành phần trang quản trị
├── lib/
│   ├── supabase/            # Kết nối Supabase (server, client, admin)
│   ├── queries/             # Hàm đọc dữ liệu
│   ├── actions/             # Server Actions (gửi form, lưu dữ liệu)
│   └── validation/          # Kiểm tra dữ liệu đầu vào (zod)
├── messages/
│   ├── vi.json              # Chuỗi giao diện tiếng Việt
│   └── en.json              # Chuỗi giao diện tiếng Anh
├── supabase/
│   ├── migrations/          # Định nghĩa bảng và phân quyền
│   └── seed.sql             # Dữ liệu mẫu để phát triển
└── docs/
```

## 5. Thiết kế dữ liệu

Thiết kế theo danh sách bảng trong `README.md`, bổ sung vài bảng phụ cần cho MVP. Nội dung có thể dịch (tên, mô tả) lưu dạng JSON theo ngôn ngữ, ví dụ `{"vi": "...", "en": "..."}`, để thêm tiếng Trung sau này chỉ cần thêm khóa `zh`.

### Bảng chính

| Bảng | Trường quan trọng | Ghi chú |
| --- | --- | --- |
| `areas` | `slug`, `name`, `city`, `district` | Khu vực, dùng cho bộ lọc và landing page. Bổ sung so với đặc tả. |
| `properties` | `slug`, `name`, `address`, `area_id`, `lat`, `lng`, `description`, `rules`, `manager_contact`, `status` (`active`, `inactive`), `is_featured` | Tòa nhà hoặc khu lưu trú. |
| `unit_types` | `slug`, `name`, `category` | Phòng trọ có nội thất, studio, căn hộ 1 phòng ngủ... |
| `units` | `slug`, `code`, `property_id`, `unit_type_id`, `title`, `description`, `area_m2`, `bedrooms`, `bathrooms`, `floor`, `furnishing` (`full`, `partial`, `none`), `status`, `is_published`, `is_featured` | Phòng hoặc căn hộ. |
| `unit_prices` | `unit_id`, `term` (`daily`, `monthly`, `long_term`), `price`, `min_term`, `deposit`, `service_fee`, `management_fee` | Giá theo thời hạn thuê. Bổ sung so với đặc tả. |
| `unit_fees` | `unit_id`, `type` (`electricity`, `water`, `internet`, `parking`, `other`), `amount`, `unit_label` (ví dụ "/kWh", "/người/tháng") | Chi phí điện, nước, internet, gửi xe minh bạch. Bổ sung so với đặc tả. |
| `amenities` | `slug`, `name`, `icon`, `scope` (`unit`, `property`) | Tiện nghi trong phòng hoặc tiện ích tòa nhà. |
| `unit_amenities` | `unit_id`, `amenity_id` | Tiện nghi của từng phòng. |
| `property_amenities` | `property_id`, `amenity_id` | Tiện ích chung của tòa nhà. Bổ sung so với đặc tả. |
| `nearby_places` | `property_id`, `name`, `type` (`school`, `office`, `hospital`, `center`), `distance_m` | Khoảng cách đến địa điểm xung quanh. Bổ sung so với đặc tả. |
| `media` | `owner_type` (`property`, `unit`), `owner_id`, `kind` (`image`, `video`, `tour_360`), `storage_path` hoặc `url`, `alt`, `sort_order`, `is_cover` | Hình ảnh và video. |
| `customers` | `full_name`, `phone`, `email`, `nationality`, `preferred_language`, `status` | Khách hàng, gộp theo số điện thoại. |
| `leads` | `customer_id`, `unit_id`, `property_id`, `type` (`consult`, `viewing`), `message`, `desired_move_in`, `desired_term`, `budget`, `preferred_viewing_at`, `source`, `utm_*`, `status`, `assigned_to` | Yêu cầu từ website. Lưu nguồn khách để báo cáo. |
| `viewing_appointments` | `lead_id`, `unit_id`, `scheduled_at`, `staff_id`, `status` | Tạo sẵn; giao diện đầy đủ ở giai đoạn 2. |
| `contracts` | `unit_id`, `customer_id`, `start_date`, `end_date`, `rent`, `deposit`, `status` | Tạo sẵn cho giai đoạn 3. |
| `articles` | `slug`, `title`, `content`, `cover`, `seo`, `published_at` | Bài viết, tin tức. |
| `content_blocks` | `key`, `value` | Banner, FAQ, thông tin liên hệ, nội dung trang giới thiệu, có thể sửa không cần lập trình. Bổ sung so với đặc tả. |
| `users` | `id` (liên kết Supabase Auth), `full_name`, `role`, `is_active` | Tài khoản nội bộ. |
| `user_properties` | `user_id`, `property_id` | Gán nhân viên hoặc quản lý phụ trách tòa nhà. Bổ sung so với đặc tả. |
| `audit_logs` | `user_id`, `action`, `entity`, `entity_id`, `changes`, `created_at` | Lịch sử thao tác, ghi tự động bằng trigger. |

### Trạng thái phòng

`available` (còn trống), `reserved` (đã giữ chỗ), `occupied` (đang thuê), `maintenance` (bảo trì). Phòng chỉ hiện ra ngoài khi `is_published = true`; phòng đang thuê vẫn có thể hiện với nhãn "Đã cho thuê" hoặc ẩn đi, tùy cấu hình.

### Trạng thái yêu cầu tư vấn

`new` (mới) → `contacted` (đã liên hệ) → `viewing_scheduled` (đã hẹn xem) → `viewed` (đã xem) → `won` (đã chốt) hoặc `lost` (không chốt).

## 6. Phân quyền

Phân quyền thực thi ngay trong cơ sở dữ liệu bằng Row Level Security của Supabase, nên kể cả khi gọi API trực tiếp cũng không vượt quyền.

| Vai trò | Mã | Quyền trong MVP |
| --- | --- | --- |
| Khách truy cập | (không đăng nhập) | Chỉ đọc phòng và tòa nhà đã công khai; chỉ được tạo yêu cầu tư vấn qua form. |
| Nhân viên tư vấn | `sales` | Xem và cập nhật yêu cầu tư vấn được giao hoặc thuộc tòa nhà phụ trách. |
| Quản lý tòa nhà | `property_manager` | Sửa phòng, giá, tình trạng, hình ảnh của tòa nhà được gán; xem yêu cầu của tòa nhà đó. |
| Quản trị viên | `admin` | Toàn quyền: tòa nhà, phòng, nội dung, người dùng. |
| Chủ doanh nghiệp | `owner` | Xem toàn bộ dữ liệu và báo cáo; không cần quyền sửa. |

## 7. Luồng tiếp nhận khách

1. Khách điền form trên trang chi tiết phòng, trang tòa nhà hoặc trang liên hệ (họ tên, số điện thoại, email, nhu cầu, thời gian muốn xem).
2. Máy chủ kiểm tra dữ liệu, chống spam, ghi nhận nguồn khách (UTM, trang gửi form).
3. Hệ thống tìm khách cũ theo số điện thoại hoặc tạo khách mới, rồi tạo yêu cầu với trạng thái `new`.
4. Gửi email báo cho nhân viên phụ trách tòa nhà (hoặc hộp thư chung nếu chưa gán) và email xác nhận cho khách nếu khách có email.
5. Nhân viên mở `/admin/leads`, gọi cho khách và cập nhật trạng thái.
6. Sự kiện `generate_lead`, bấm gọi điện, bấm Zalo được gửi lên Google Analytics để đo chuyển đổi.

## 8. SEO và hiệu năng

- Trang công khai render sẵn phía server và làm mới theo chu kỳ hoặc ngay khi quản trị viên lưu thay đổi.
- Mỗi phòng, tòa nhà, khu vực có tiêu đề, mô tả, ảnh chia sẻ (Open Graph) riêng; quản trị viên sửa được.
- Dữ liệu có cấu trúc JSON-LD: `Organization`, `LocalBusiness`, `Apartment`/`Accommodation`, `BreadcrumbList`, `FAQPage`.
- `sitemap.xml` tự sinh từ dữ liệu, có thẻ `hreflang` cho từng ngôn ngữ; `robots.txt` chặn `/admin`.
- Ảnh tự nén sang WebP/AVIF, tải chậm ảnh ngoài màn hình.
- Mục tiêu: điểm Lighthouse trên điện thoại từ 90 trở lên cho hiệu năng, SEO và khả năng truy cập.

## 9. Các bước triển khai

Mỗi bước kết thúc bằng một PR có bản xem thử trên Vercel để duyệt.

| Bước | Nội dung | Kết quả kiểm tra được |
| --- | --- | --- |
| 1. Khởi tạo dự án | Next.js, TypeScript, Tailwind, shadcn/ui, ESLint, Prettier, `next-intl`, CI chạy lint và build. | Trang trống chạy được, đổi được `/vi` và `/en`. |
| 2. Cơ sở dữ liệu | Viết migration cho các bảng ở mục 5, RLS ở mục 6, trigger `audit_logs`, dữ liệu mẫu (khoảng 3 tòa nhà, 20 phòng). | Chạy Supabase local, truy vấn được dữ liệu mẫu, kiểm tra quyền khách không sửa được dữ liệu. |
| 3. Bộ khung giao diện | Màu, font, header, footer, menu điện thoại, thanh nút gọi/Zalo/đăng ký xem cố định. | Xem trên điện thoại thấy nút liên hệ luôn hiển thị. |
| 4. Danh sách và chi tiết phòng | Trang danh sách với bộ lọc và sắp xếp; trang chi tiết với ảnh, giá, các khoản phí, tiện nghi, bản đồ, khoảng cách, tình trạng. | Lọc theo khu vực, giá, diện tích, loại, số phòng ngủ, tiện nghi cho kết quả đúng. |
| 5. Tòa nhà và khu vực | Trang tòa nhà, landing page khu vực và khu vực theo loại phòng. | Mỗi tòa nhà, khu vực có URL và metadata riêng. |
| 6. Form và nhận khách | Form tư vấn và đặt lịch xem, chống spam, email thông báo. | Gửi form, thấy yêu cầu trong cơ sở dữ liệu và nhận được email. |
| 7. Trang nội dung | Trang chủ, giới thiệu, hướng dẫn thuê, câu hỏi thường gặp, liên hệ; nội dung lấy từ `content_blocks`. | Sửa nội dung trong cơ sở dữ liệu, website cập nhật theo. |
| 8. Trang quản trị | Đăng nhập; quản lý tòa nhà, phòng, giá, phí, tiện nghi, ảnh (tải lên, sắp xếp, chọn ảnh bìa); quản lý yêu cầu; quản lý nội dung; quản lý người dùng. | Quản trị viên thêm, sửa, ẩn phòng mà không cần lập trình viên. |
| 9. SEO và đo lường | Metadata, JSON-LD, sitemap, `hreflang`, GA4 và sự kiện chuyển đổi, Search Console. | Công cụ kiểm tra dữ liệu có cấu trúc của Google không báo lỗi; GA4 ghi nhận sự kiện. |
| 10. Kiểm thử và ra mắt | Kiểm thử trên điện thoại và máy tính, đo Lighthouse, nhập dữ liệu thật, gắn tên miền và SSL. | Đạt toàn bộ tiêu chí ở mục 13 của `README.md`. |

Bước 1 đến 4 và bước 6 có thể làm ngay với dữ liệu mẫu, không phải chờ thương hiệu hay dữ liệu thật. Bước 7 và 10 cần nội dung chính thức của công ty.

## 10. Những điều cần chốt

Các mục sau không chặn việc bắt đầu lập trình, nhưng cần trả lời trước bước tương ứng:

| Câu hỏi | Ảnh hưởng đến bước | Đề xuất mặc định |
| --- | --- | --- |
| Tên thương hiệu, logo, màu nhận diện | 3 | Tạm dùng "MiaHouse" và bảng màu trung tính, thay sau. |
| Danh sách khu vực hoạt động (quận/thành phố) | 2, 5 | Tạo khu vực mẫu tại TP.HCM. |
| Phòng đã cho thuê: ẩn hẳn hay hiện nhãn "Đã cho thuê" | 4 | Hiện nhãn để tăng nội dung SEO, không cho đặt lịch. |
| Đơn vị tiền tệ hiển thị cho khách nước ngoài | 4 | Chỉ VNĐ trong MVP. |
| Email nhận thông báo khách mới | 6 | Một hộp thư chung, sau đó gán theo tòa nhà. |
| Số Zalo, Zalo OA hay số cá nhân | 3, 6 | Liên kết `zalo.me/<số điện thoại>`. |
| Có cần WhatsApp ngay từ đầu không | 3 | Có, chỉ hiện ở giao diện tiếng Anh. |
| Tài khoản Supabase, Vercel, tên miền đứng tên ai | 1, 10 | Tạo dưới tài khoản công ty, cấp quyền cho nhóm phát triển. |

## 11. Rủi ro và cách xử lý

| Rủi ro | Cách xử lý |
| --- | --- |
| Ảnh thật chưa có hoặc chất lượng thấp | Dùng ảnh mẫu khi phát triển; chuẩn bị hướng dẫn chụp ảnh và kích thước tối thiểu. |
| Giá và tình trạng phòng không được cập nhật kịp | Hiện "Cập nhật lần cuối" trên trang phòng; trang tổng quan quản trị cảnh báo phòng lâu chưa cập nhật. |
| Spam form | Turnstile, giới hạn số lần gửi theo IP và số điện thoại. |
| Nội dung tiếng Anh chưa dịch đủ | Trang chưa có bản dịch thì không xuất hiện ở `/en` và không vào sitemap tiếng Anh. |
| Dữ liệu cá nhân của khách | Chỉ nhân sự có quyền mới xem được; bổ sung chính sách bảo mật và ô đồng ý trên form theo quy định về bảo vệ dữ liệu cá nhân. |
