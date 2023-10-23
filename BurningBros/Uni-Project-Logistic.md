## Domain: Logistic

Having 7 table
High module: Bang Ke
# Invoice table
- ### Invoice list
	- Data structure:
		- Invoice-no
		- Invoice-date
		- Contract-no
- ### Product relate
	- Data structure:
		- 
# PSR
- Function:
- Data structure:
	- HS code
	- Form: 
		- Ex: AK, AJ, VJ, AANZ, AHK, CPTPP, VC, VK, RCEP, AC, EUEA, EV, UKV, ATIGA
# C/O list
- Function:
	- Import
	- Read
- Data structure: 
	- Reference-no
	- Issue-date
	- Export-declare-number
	- Part-number: <Mã nguyên liệu, vật tư>
# ANNEX10
- Function: 
	-  Import
	- Read
- Data structure:
	- Exporter
	- Import declare number: <Số TK>
	- Part-number: <Mã nguyên liệu, vật tư>
# Register stock
# ECUS
- Function:
	- Read
	- Update
	- Delete
- Structure:
	1. Register-form: Số TK
	2. Register-date: Ngày đk
	3. Item-code: Mã NPL/SP
	4. HS: Mã HS
	5. Unit: Đơn giá
	6. Đơn vị tính
	7. Tổng số lượng
	8. Số hoá đơn
	9. Ngày hoá đơn
	10. Số hợp đồng
	11. Ngày hợp đồng
# BOM
- Function: 
	- Read
	- Update
	- Delete

# Module Bang Ke
- ## Lookup for BANGKE
	1. Using product code on the ECUS table to lookup material code on the BOM table, then store material name, unit and BOM value to BANGKE
	   From material code on the BOM table lookup back to the ECUS table to pick HS code and import country
	    => BANGKE have: **Material name - HS code - unit - BOM value - country**
	2. Using material code (lookup in step 1) to find CIF and Customers Declaration date//Number on the ECUS table then using Start_date (a) and End date to filter the CD & CIF
	   => We have the list of declaration number, next we lookup for the origin
	3. **Deduction origin** 
	   _ Using material code (NVL/SP) and declaration number (SoTK) to lookup on the C/O list table. 
	   _ If there is any data which mapped with the material code and declaration number.
	   => return "Reference No" & "Issued date" to declaration number and date of BANGKE
	   _ Else there is no mapping data on the C/O list => using material code (NVL/SP) and declaration number (SoTK) to lookup on the ANNEX10 table
		   _ If there is any data which mapped with the material code and declaration number
		   => return the "Exporter" to declaration number of the BANGKE
		   _ Else => The material do not have any origin
	4. Deduction
		1. Calculate the total material quantity = BOM value * total (b)
		2. Finding the CD number (**Finding on ECUS table**)
			- Lookup for the declaration which have the origin first (step3) (c)
				1. Finding the declaration which have the stock higher than the total material quantity (b)
				2. Finding the declaration which have the lowest price
				3. Finding the declaration which have the "NgayTK" near "start_date" (a)
			- If the material do not have CD number (c)
				1. Continue search the CD number without origin
				2. Finding the declaration which have the stock higher than the total material quantity (b)
				3. Finding the declaration which have the lowest price
				4. Finding the declaration which have the "NgayTK" near the "start_date" (a)
			- => Fill the "Đơn giá", "xuất xứ", "Số TK", "Ngày ĐK" if it matched (c) condition
	- ### Now BANGKE have: **Material name - HS code - unit - BOM value - country - Đơn giá - xuất xứ - Số TK - Ngày ĐK**
	  
	  
- ## Result of the BANGKE 
	1. We use the "form_type" from the beginning to check the result
		1. CTC: Lookup the HS code to check if the result is valid or not
		2. RVC: use formula to calculate the result to check if the result is valid or not
		- => If the form_type is RVC and the result is not valid => recalculate the data
		- a
		----------------------------------------------------------------------
		1.  IF the form_type == CTC: 
			1. Check HS code based on "criteria to apply":
				- CC: ------- Lookup based on 2 first HS code
				- CTH: ----- Lookup based on 4 first HS code
				- CTSH: ---- Lookup based on 6 first HS code
			2. IF HS code of material different with HS code of product 
			   => Result is valid
		-----------------------------------------------------------------------
		2. If form_type == RVC:
			1. Check Giá is FOB or EXW to know the formula
			   - Data = Giá FOB 
			   - => Use formula: RS = (FOB - sum of material without origin) / FOB
			   ----------------------------------------------------
			   - Data = Giá EXW
			   - => Use formula: Sum of material without origin / EXW
			   ----------------------------------------------------
			   - Data = Giá EXW & "user_input" type is progress price
			   - => Use formula: RS = Trị giá NVL đầu vào ko xx VN / Trị giá xuất xưởng * 100
			2. Check the condition based on "criteria to apply"
			   + RVC{%}: Use the value of the RVC to compare with the result
			3. If RS is lower than condition, result is valid
			   => Add text: "Kết luận: hàng hoá {Đáp ứng/Không đáp ứng} tiêu chí {criteria to apply}"
			4. Else: Re-calculate the BANGKE
				1. Recheck the data on the deduction logic (STEP 4 Lookup for BANGKE)
				2. Find another register_account with lower price, have higher stock and near the start date.
		-------------------------------------------------------------------------




Improvement:
1. Dùng product code và contract number trên ECUS để tìm mã NVL/SP trên BOM table (nếu form là EUR1 hoặc EUR1.UK thì tìm trên BOM-EXW) 
2. Sau khi có mã NVL/SP sẽ lấy nó và contract number để tìm ngược lại trên ECUS hoặc PURCHASE (Dựa vào note Nếu note là **NKTC** => ecus(origin=VN) , **NK** => Ecus (origin!=VN) , **VN** => purchase , empty => both)
	- Chỉ áp dụng khi lookup trên purchase: 
		1. Nếu register_form của purchase match với import declaration number trên annex10 
		   => origin = Viet Nam
		2. Nếu register_form của purchase không match với import declaration number trên annex10 => origin = KHÔNG XX 

Ms Lam and Ms Trang is on testing bangke feature, i see they confuse in how to test they dont' know how to start, i think they need a video or an user guide to instruct them how to use this feature.

About current status, yesterday they provide we an additional logic to improve the DB and to lookup data across table. Mr Ân have provide it to dev team and they on develop it, now they still on test 

_ Our system is working completely
_ We just modified some logic to lookup data on the BANGKE function 
_ I'm following up for next requirement of UNI team
_ Uni team is on testing BANGKE function and i think they need user guide or video or a training session to use our system 

![[Pasted image 20231019170802.png]]

