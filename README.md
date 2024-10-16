![giris](https://github.com/user-attachments/assets/1772c79c-33c2-4cb6-9abf-92f4bf007ef7)

# NORTHWIND Dataseti SQL Analizi

## 1. ÜRÜN PERFORMANS ANALİZİ:
## BELİRLİ BİR ZAMAN DİLİMİNDEKİ SATIŞ TRENDLERİNİ GÖZLEMLEMEK.

SELECT
	TEXT(DATE_PART('year', O.ORDER_DATE)) 
		|| '-' || 	
		CASE WHEN DATE_PART('month', O.ORDER_DATE) < 10 
			THEN '0' || TEXT(DATE_PART('month', O.ORDER_DATE))
		ELSE TEXT(DATE_PART('month', O.ORDER_DATE))
		END
	AS DATE_FORMATED,
	SUM(D.QUANTITY),
	ROUND(SUM(D.UNIT_PRICE * D.QUANTITY)) AS SATIS_BEDELI
FROM
	ORDERS AS O
	INNER JOIN ORDER_DETAILS AS D ON O.ORDER_ID = D.ORDER_ID
WHERE
	DATE_PART('year', O.ORDER_DATE) = 1997
GROUP BY
	DATE_FORMATED
ORDER BY
	DATE_FORMATED

![Yillik_urun_satis_bedeli](https://github.com/user-attachments/assets/33d39651-e24b-475a-85c9-2dc76d0983bd)

ANALİZ YORUMU : Bu aşamada sadece 1997 yılındaki satış trendlerini inceledim. Çünkü datada diğer yıllara ait bütün ayları kapsayan tarihler mevcut değil.
Bu aşamada görüyoruzki; satışlarda anormal bir artış yada düşüş yok. Bu da tarihlere göre satışların etkilenmediğini gösteriyor. Ama ilgimi çeken bir nokta var. O da şubat, mart ve haziran aylarında satış tutarlarının birbirine çok yakın olduğu. Bu aylar için daha detaylı araştırmalar yapılabilir.

## 2.	PERSONEL PERFORMANSI ANALİZİ:
## HANGİ ÇALIŞANLARIN DAHA FAZLA SATIŞ GERÇEKLEŞTİRDİĞİNİ VEYA DAHA YÜKSEK GELİR ELDE ETTİĞİNİ ANALİZ ETMEK.

WITH
	CALISAN_SIPARISLERI AS (
		SELECT
		E.EMPLOYEE_ID,
		E.FIRST_NAME || ' ' || E.LAST_NAME AS CALISAN_ADI,
		(D.UNIT_PRICE * QUANTITY) - ((D.UNIT_PRICE * QUANTITY) * DISCOUNT) AS SATIS_BEDELI
		FROM ORDERS AS O
			INNER JOIN ORDER_DETAILS AS D ON D.ORDER_ID = O.ORDER_ID
			INNER JOIN EMPLOYEES AS E ON E.EMPLOYEE_ID = O.EMPLOYEE_ID
		WHERE REPORTS_TO IS NOT NULL
	)
SELECT
	C.CALISAN_ADI,
	COUNT(DISTINCT O.ORDER_ID) AS SIPARIS_SAYISI,
	AVG(DISCOUNT) AS ORTALAMA_INDIRIM,
	ROUND(SUM(C.SATIS_BEDELI)) AS KAZANC
FROM
	ORDERS AS O
	INNER JOIN ORDER_DETAILS AS D ON D.ORDER_ID = O.ORDER_ID
	INNER JOIN CALISAN_SIPARISLERI AS C ON C.EMPLOYEE_ID = O.EMPLOYEE_ID
GROUP BY CALISAN_ADI
ORDER BY KAZANC DESC

![calisanlarin_siparis_sayilari](https://github.com/user-attachments/assets/64b78518-432e-4a5b-b5d4-ad1554b1caeb)

![calisanlarin_ortalama_indirim_oranlari](https://github.com/user-attachments/assets/0613d63e-abef-4d95-a344-da2b4e178113)

ANALİZ YORUMU : Bu aşamada görüyoruzki sipariş sayısı ve indirim oranları arasında bir ilişki yok. Yani en çok indirim yapan  Robert King olmasına rağmen, aldığı sipariş sayıları orta seviyelerde kalıyor. 
Ama genel olarak görüyoruzki indirim oranları bütün satıcılar için yüksek. Bu durumda neden bu kadar çok indirim yapıldığının araştırılması gerekmektedir. Gereksiz yere yapılan indirimler varsa, bunların düzeltilerek kazancın artması sağlanabilir.

