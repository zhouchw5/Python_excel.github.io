# Some Basic Usages of Python in My Daily work           
- **Still with my original insistence, your footprint in my website would not be marked or stored. I always wish a zero intersection with me after your reading, nothing but some interesting memory merely in your mind, no storage for the Like or reader amount. Your ever presence is enough.**             


My main role in HUAWEI was a supply chain management engineer, or specifically a global master planner of IT products, to achieve the best strategy in demand fulfillment and cost control. My main job can be divided into three modules: Demand/Supply Match, Logistics Optimization and Allocations Management, all of which were mostly based on some enterprise-level planning systems like Advanced Planning System, MRP, MRP II, ERP, ISC+, etc. Though some systematic processes have been developed for our daily decision-making, some personal capabilities like programming via Python or VBA also helped simplify analysing and communicating procedures. And for my further occupational career in data analysis, in this gap year I also joined some programming projects led by Doc. Lin to develop some interaction and visualization interfaces for data report. This article would share some basic techniques in Python's interaction with Excel which I have used in some actual scenarios. Due to information security, I would only use some fictitious and simplified data for examples illustration if essential.         
          
          
Yours,         
Zhou Mr. Chuwei          
2018.08.08       

------------------------                   
         
## Scientific Python Development Environment               
I strongly recommend that we should download the open source environment of Python, the Anaconda Navigator, which contains more than one hundred installed toolkits in Python, like numpy, pandas, os, etc.       
![anaconda](https://github.com/zhouchw5/Python_excel.github.io/blob/master/anaconda.png)              
_Spyder is a powerful integrated development environment with advanced editing, interactive testing, debugging and introspection features._          
          
------------------------           
             
## Actual Cases           
_First part for some important modules we should import before programming, like os, pandas as pd, numpy as np, and datetime._         
- The os in my programme is mainly used to join selected files to the defined project path.                
- The numpy is mainly used for the data selection and analysis.        
- We would also import Workbook from openpyxl and import dataframe_to_rows from openpyxl.utils.dataframe. The right mix of pandas module and openpyxl help interact with xlsx. files via the transmission of data effectively.                
         
### Interface With Excel        
_Here's the convenience to interaction with Excel files via Python, which is also the fundamental bridge to remove data from Excel for further computing in Python._                   
``` python         
proj_folder = r'D:\Drivers\python_test'
def read_excel(proj_folder_path, file_name, sheetname):
    current_file = os.path.join(proj_folder_path, file_name)
    table = pd.read_excel(current_file, sheet_name = sheetname)
    table.columns = map(str.lower, table.columns)
    return table          
test_01 = read_excel(proj_folder, 'test_file', 'test_sheet')         
```       
The coding above is my customary option to read an xlsx. file with defining a function read_excel, via using the module pandas. Other methods like importing the module xlrd could also be used to grab the data in an xlsx. file.           











