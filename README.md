# ---Exploratory-Data-Analysis
SELECT*
FROM layoffs_staging2;

-- phân tích là lọc các dữ liệu cần cho doanh nghiệp
SELECT MAX(total_laid_off),MAX(percentage_laid_off)
FROM layoffs_staging2;

SELECT*
FROM layoffs_staging2
WHERE percentage_laid_off = 1
order by total_laid_off Desc; # Sắp xếp kết quả theo số lượng nhân viên bị sa thải  giảm dần, tức là công ty sa thải nhiều nhân viên nhất sẽ được xếp lên đầu

SELECT*
FROM layoffs_staging2
WHERE percentage_laid_off = 1
# Sắp xếp danh sách theo số tiền huy động được (funds_raised_millions) giảm dần, nghĩa là công ty nào huy động được nhiều tiền nhất sẽ đứng đầu danh sách.
order by  funds_raised_millions Desc;
#Chọn cột company (tên công ty) và tính tổng số nhân viên bị sa thải (SUM(total_laid_off)) của từng công ty.
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
group by company 
order by 2 desc;#giảm dần, có nghĩa là công ty nào sa thải nhiều nhân viên nhất sẽ được xếp lên đầu danh sách.
#việc sa thải bắt đầu từ tg nào kết thúc thời gian nào
SELECT MIN(date),MAX(date)
FROM layoffs_staging2;
#giảm dần, nghĩa là ngành nào có tổng số nhân viên bị sa thải cao nhất sẽ xuất hiện đầu tiên.
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
group by industry
order by 2 desc;
#giảm dần, nghĩa là quốc gia nào có tổng số nhân viên bị sa thải cao nhất sẽ xuất hiện đầu tiên.
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
group by country
order by 2 desc;
SELECT *
FROM layoffs_staging2;
#ngày có lượng sa thải nhân viên lớn nhất
SELECT date, SUM(total_laid_off)
FROM layoffs_staging2
group by date
order by 2 desc;
#năm có lượng nhân viên sa thải lớn nhất
SELECT year(date), SUM(total_laid_off)
FROM layoffs_staging2
group by year(date)
order by 2 desc;
#Truy vấn này giúp phân tích xem công ty ở giai đoạn phát triển nào (ví dụ: Startup, Growth, Public,...) bị ảnh hưởng nặng nề nhất bởi tình trạng sa thải.
SELECT stage, SUM(total_laid_off)
FROM layoffs_staging2
group by stage
order by 2 desc;
#câu lệnh SQL này tính tổng tỷ lệ sa thải
SELECT company, SUM(percentage_laid_off)
FROM layoffs_staging2
group by company
order by 2 desc;
#câu lệnh SQL này tính trung bình tỷ lệ sa thải
SELECT company, AVG(percentage_laid_off)
FROM layoffs_staging2
group by company
order by 2 desc;
#Kết quả truy vấn sẽ là danh sách các tháng
SELECT substring(date,6,2) AS 'MOTH', SUM(total_laid_off)
FROM layoffs_staging2
group by  substring(date,6,2) 
order by 2 desc;
# tính tổng số nhân viên bị sa thải theo từng tháng
SELECT substring(date,1,7) AS MOTH, SUM(total_laid_off)
FROM layoffs_staging2
where  substring(date,1,7) IS NOT NULL#Lọc ra các giá trị date không rỗng để tránh lỗi khi nhóm dữ liệu theo tháng.
group by MOTH 
order by 1 asc;



SELECT*
FROM layoffs_staging2;
#Truy vấn này giúp tính tổng cộng dồn số nhân viên bị sa thải theo thời gian, giúp phân tích xu hướng sa thải tích lũy.
WITH Rolling_TOTAL AS
(SELECT substring(date,1,7) AS MOTH, SUM(total_laid_off) AS total_off
FROM layoffs_staging2
where  substring(date,1,7) IS NOT NULL
group by MOTH 
order by 1 asc)
select MOTH,total_off,sum(total_off) OVER (order by MOTH) AS Rolling_TOTAL
FROM Rolling_TOTAL;
#Truy vấn này giúp bạn nhanh chóng thấy được tổng số nhân viên bị sa thải theo từng năm của mỗi công ty
SELECT company,YEAR(date), sum(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(date)
order by 3 desc;#Sắp xếp theo cột thứ 3

SELECT company,YEAR(date), sum(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(date)
order by 3 desc;



#
With Company_year(company,year,total_laid_off) AS
(SELECT company,YEAR(date), sum(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(date))#nếu có with không dùng thêm order
SELECT *  
FROM Company_year  
ORDER BY total_laid_off DESC;
# chuyển qua
#để tổng hợp dữ liệu về số lượng nhân viên bị sa thải theo từng năm, sau đó sử dụng DENSE_RANK() để xếp hạng các công ty dựa trên tổng số nhân viên bị sa thải mỗi năm.
With Company_year(company,year,total_laid_off) AS
(SELECT company,YEAR(date), sum(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(date))#nếu có with không dùng thêm order
SELECT * , dense_rank() over(partition by year order by total_laid_off desc) AS ranking#DENSE_RANK() OVER (...):Xếp hạng công ty theo số lượng nhân viên bị sa thải trong từng năm.
FROM Company_year;
# lọc null
With Company_year(company,year,total_laid_off) AS
(
SELECT company,YEAR(date), sum(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(date))#nếu có with không dùng thêm order
SELECT * , dense_rank() over(partition by year order by total_laid_off desc) AS ranking
FROM Company_year
where year is not null;
# tinh gọn
With Company_year(company,year,total_laid_off) AS
(
SELECT company,YEAR(date), sum(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(date))#nếu có with không dùng thêm order
SELECT * , dense_rank() over(partition by year order by total_laid_off desc) AS ranking
FROM Company_year
where year is not null
order by ranking ASC;#🔹 ORDER BY ranking ASC; giúp sắp xếp công ty theo thứ hạng từ thấp đến cao (công ty sa thải nhiều nhất trong năm đứng đầu danh sách
# tin gọn
With Company_year(company,year,total_laid_off) AS
(
SELECT company,YEAR(date), sum(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(date)),Company_year_rank AS #nếu có with không dùng thêm order
(SELECT * , dense_rank() over(partition by year order by total_laid_off desc) AS ranking
FROM Company_year
where year is not null
order by ranking ASC) 
select*
from  Company_year_rank;
# tạo bảng company rank
CREATE TABLE company_year_rank AS 
WITH Company_year AS (
    SELECT company, YEAR(date) AS year, SUM(total_laid_off) AS total_laid_off
    FROM layoffs_staging2
    GROUP BY company, YEAR(date)
)
SELECT *, DENSE_RANK() OVER (PARTITION BY year ORDER BY total_laid_off DESC) AS ranking
FROM Company_year;
# chạy lệnh
select*
from Company_year_rank 
where ranking >= 5;
