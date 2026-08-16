# ROUND 2 — Resort Booking Dataset (Questions + Answers)

3 files: **bookings.csv** (360 rows, messy), **guests.csv** (60 guests), **rooms.csv** (5 room types)

```python
import pandas as pd
bookings = pd.read_csv('bookings.csv')
guests = pd.read_csv('guests.csv')
rooms = pd.read_csv('rooms.csv')
```

> **Tarika:** Pehle khud solve karo (answer chhupa ke), fir neeche check karo. Ye Round 1 se thoda tough h — kuch NAYE concepts bhi h (marked with 🆕).

---

## SECTION 1: Cleaning

**Q1. Har column me kitni missing values h check karo.**
```python
bookings.isnull().sum()
```

**Q2. `Source` column clean karo (spacing/case issues hai — 'Direct', 'direct', ' Direct' sab same hona chahiye).**
```python
bookings['Source'] = bookings['Source'].str.strip().str.title()
```

**Q3. `guests` table me City clean karo.**
```python
guests['City'] = guests['City'].str.strip().str.title()
```

**Q4. Duplicate bookings dhundo aur hatao.**
```python
print(bookings.duplicated().sum())
bookings = bookings.drop_duplicates()
```

**Q5. `Nights` ki missing values ko median se bharo.**
```python
bookings['Nights'] = bookings['Nights'].fillna(bookings['Nights'].median())
```

**Q6. `GuestsCount` ki missing values ko us RoomType ke mode (sabse common value) se bharo. 🆕**
```python
bookings['GuestsCount'] = bookings.groupby('RoomType')['GuestsCount'].transform(
    lambda x: x.fillna(x.mode()[0] if not x.mode().empty else x.median())
)
```
Explanation: Mode = sabse zyada baar aane wali value. Har RoomType ke apne guests count ka common pattern hota h (jaise Dormitory me zyada log rukte h), isliye group-wise mode use kiya.

**Q7. `TotalAmount` ki missing values ko `Nights * RoomType ka Base_Price` se recalculate karo (rooms table se merge karke).**
```python
bookings = pd.merge(bookings, rooms[['RoomType','Base_Price']], on='RoomType', how='left')
bookings['TotalAmount'] = bookings['TotalAmount'].fillna(bookings['Nights'] * bookings['Base_Price'])
```

**Q8. `Rating` ki missing values ko us RoomType ke average rating se bharo.**
```python
bookings['Rating'] = bookings.groupby('RoomType')['Rating'].transform(lambda x: x.fillna(x.mean()))
```

---

## SECTION 2: GroupBy, Merge, Pivot (Advanced)

**Q9. Har RoomType ka total revenue (TotalAmount) nikalo, sabse zyada wale se sort karo.**
```python
bookings.groupby('RoomType')['TotalAmount'].sum().sort_values(ascending=False)
```

**Q10. Ek sath multiple stats nikalo: har Source ka total bookings count, average TotalAmount, aur average Rating. 🆕**
```python
bookings.groupby('Source').agg(
    Total_Bookings=('BookingID', 'count'),
    Avg_Amount=('TotalAmount', 'mean'),
    Avg_Rating=('Rating', 'mean')
).sort_values('Total_Bookings', ascending=False)
```
Explanation: `agg()` me dictionary jaisa naam de kar ek sath multiple alag-alag column pe alag function chala sakte h — ye real reports banane me bahut kaam aata h.

**Q11. `bookings` ko `guests` ke sath merge karo (GuestID se).**
```python
full_data = pd.merge(bookings, guests, on='GuestID', how='left')
```

**Q12. Har City (guest ki city) se kitna total revenue aaya, nikalo.**
```python
full_data.groupby('City')['TotalAmount'].sum().sort_values(ascending=False)
```

**Q13. Pivot table banao: rows me RoomType, columns me Source, values me total TotalAmount.**
```python
pd.pivot_table(full_data, values='TotalAmount', index='RoomType', columns='Source', aggfunc='sum', fill_value=0)
```

**Q14. Top 5 guests kaun h jo sabse zyada kharch karte h.**
```python
full_data.groupby('GuestName')['TotalAmount'].sum().sort_values(ascending=False).head(5)
```

**Q15. Har guest kitni baar aaya (booking count), sabse zyada aane wale 5 guests dikhao. 🆕**
```python
full_data.groupby('GuestName')['BookingID'].count().sort_values(ascending=False).head(5)
```

---

## SECTION 3: Date Analysis

**Q16. `CheckinDate` ko datetime me convert karo, aur usse Month aur Weekday (Monday/Tuesday etc.) nikalo. 🆕**
```python
bookings['CheckinDate'] = pd.to_datetime(bookings['CheckinDate'])
bookings['Month'] = bookings['CheckinDate'].dt.month
bookings['Weekday'] = bookings['CheckinDate'].dt.day_name()
```

**Q17. Kaunse Weekday (Saturday/Sunday etc.) pe sabse zyada bookings hoti h?**
```python
bookings['Weekday'].value_counts()
```

**Q18. Month-wise total revenue trend nikalo.**
```python
bookings.groupby('Month')['TotalAmount'].sum()
```

---

## SECTION 4: Visualization

**Q19. RoomType-wise total revenue ka bar chart.**
```python
import seaborn as sns, matplotlib.pyplot as plt
sns.barplot(x='RoomType', y='TotalAmount', data=bookings, estimator=sum)
plt.show()
```

**Q20. Month-wise revenue trend ka line chart.**
```python
monthly = bookings.groupby('Month')['TotalAmount'].sum().reset_index()
sns.lineplot(x='Month', y='TotalAmount', data=monthly)
plt.show()
```

**Q21. `TotalAmount` ka boxplot — outliers dhundo (4 jaan-bujh kar dale h).**
```python
sns.boxplot(x=bookings['TotalAmount'])
plt.show()
```

**Q22. Weekday-wise bookings count ka bar chart.**
```python
sns.countplot(x='Weekday', data=bookings)
plt.show()
```

**Q23. Rating ki distribution dekhne ke liye histogram.**
```python
plt.hist(bookings['Rating'], bins=5)
plt.show()
```

---

## SECTION 5: EDA + Statistics

**Q24. "SHICVI" process poore bookings dataframe pe chalao.**
```python
bookings.shape
bookings.head()
bookings.info()
# clean (already done above)
# visualize (already done above)
# insight likho
```

**Q25. IQR method se `TotalAmount` ke outliers nikalo.**
```python
Q1 = bookings['TotalAmount'].quantile(0.25)
Q3 = bookings['TotalAmount'].quantile(0.75)
IQR = Q3 - Q1
outliers = bookings[(bookings['TotalAmount'] < Q1 - 1.5*IQR) | (bookings['TotalAmount'] > Q3 + 1.5*IQR)]
outliers
```

**Q26. Guests ko age ke basis pe categories me baato: '18-30', '31-45', '46-65'. 🆕**
```python
bins = [18, 30, 45, 65]
labels = ['18-30', '31-45', '46-65']
guests['AgeGroup'] = pd.cut(guests['Age'], bins=bins, labels=labels)
```
Explanation: `pd.cut()` numeric column ko categories (bins) me todta h — jaise Excel me IF-nested formula se age group banana, par ek line me.

**Q27. Har AgeGroup ka total spending (TotalAmount) nikalo.**
```python
full_data2 = pd.merge(bookings, guests, on='GuestID', how='left')
bins = [18, 30, 45, 65]
labels = ['18-30', '31-45', '46-65']
full_data2['AgeGroup'] = pd.cut(full_data2['Age'], bins=bins, labels=labels)
full_data2.groupby('AgeGroup')['TotalAmount'].sum()
```

**Q28. RoomType ko unke total revenue ke basis pe rank do (1 = highest revenue). 🆕**
```python
revenue = bookings.groupby('RoomType')['TotalAmount'].sum().reset_index()
revenue['Rank'] = revenue['TotalAmount'].rank(ascending=False)
revenue.sort_values('Rank')
```

---

## SECTION 6: SQL Practice

**Q29. `bookings` ko SQLite me save karo.**
```python
import sqlite3
conn = sqlite3.connect('resort.db')
bookings.to_sql('bookings_table', conn, if_exists='replace', index=False)
```

**Q30. SQL se: RoomType = 'Suite' ke saare bookings nikalo jinki Rating 4 ya usse zyada h.**
```python
pd.read_sql_query("SELECT * FROM bookings_table WHERE RoomType='Suite' AND Rating>=4", conn)
```

**Q31. SQL se: Source-wise total revenue, DESC order me.**
```python
pd.read_sql_query("""
SELECT Source, SUM(TotalAmount) as Revenue
FROM bookings_table
GROUP BY Source
ORDER BY Revenue DESC
""", conn)
```

**Q32. SQL JOIN se: un guests ka naam nikalo jinki total spending 10000 se zyada h.**
```python
guests.to_sql('guests_table', conn, if_exists='replace', index=False)
pd.read_sql_query("""
SELECT g.GuestName, SUM(b.TotalAmount) as Total_Spend
FROM bookings_table b
JOIN guests_table g ON b.GuestID = g.GuestID
GROUP BY g.GuestName
HAVING Total_Spend > 10000
ORDER BY Total_Spend DESC
""", conn)
```

---

## SECTION 7: Final Excel Report

**Q33. Ek Excel report banao jisme RoomType summary, Source summary, aur Weekday summary teen alag sheets me ho.**
```python
room_summary = bookings.groupby('RoomType')['TotalAmount'].sum().reset_index()
source_summary = bookings.groupby('Source')['TotalAmount'].sum().reset_index()
weekday_summary = bookings.groupby('Weekday')['BookingID'].count().reset_index()

with pd.ExcelWriter('Resort_Report.xlsx') as writer:
    room_summary.to_excel(writer, sheet_name='RoomType Summary', index=False)
    source_summary.to_excel(writer, sheet_name='Source Summary', index=False)
    weekday_summary.to_excel(writer, sheet_name='Weekday Summary', index=False)
```

---

## Naye concepts jo Round 2 me seekhe (🆕 wale)
| Concept | Kaam |
|---|---|
| `agg()` with named columns | Ek sath multiple stats nikalna, alag naam ke sath |
| `mode()` | Sabse common value nikalna |
| `.dt.day_name()` | Date se weekday nikalna |
| `pd.cut()` | Numeric ko categories/bins me todna |
| `.rank()` | Values ko rank dena (1st, 2nd, 3rd...) |

### Yaad rakhne ka tarika
Ek line: **"agg se multiple stats, cut se bins, rank se number do"**

---

## Ab aage kya?
- Round 2 complete hone ke baad batana — **Round 3** me tumhe bilkul real-jaisa **messy, bada dataset (500+ rows, thoda gandi date formats, text cleaning wala)** dunga jisme khud se sochna padega ki kaunsa function use karna h (questions bina hint ke honge)
- Uske baad seedha apna **real family business ka Excel data** le kar isi tarah practice karo
