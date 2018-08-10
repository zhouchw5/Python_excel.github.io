# Some Basic Usages of Python in My Daily work           
- **Still with my original insistence, your footprint in my website would not be marked or stored. I always wish a zero intersection with me after your reading, nothing but some interesting memory merely in your mind, no storage for the Like or reader amount. Your ever presence is enough.**             


My main role in HUAWEI was a supply chain management engineer, or specifically a global master planner of IT products, to achieve the best strategy in demand fulfillment and cost control. My main job can be divided into three modules: Demand/Supply Match, Logistics Optimization and Allocations Management, all of which were mostly based on some enterprise-level planning systems like Advanced Planning System, MRP, MRP II, ERP, ISC+, etc. Though some systematic processes have been developed for our daily decision-making, some personal capabilities like programming via Python or VBA also helped simplify analysing and communicating procedures. And for my further occupational career in data analysis, in this gap year I also joined some programming projects led by **Doc. Lin** to develop some interaction and visualization interfaces for data report. This article would share some basic techniques in Python's interaction with Excel which I have used in some actual scenarios. Due to information security, I would only use some fictitious and simplified data for examples illustration if essential.         
          
          
Yours,         
Zhou Mr. Chuwei          
2018.08.08       

------------------------                   
         
## Scientific Python Development Environment               
I strongly recommend that we should download the open source environment of Python, the Anaconda Navigator, which contains more than one hundred installed toolkits in Python, like numpy, pandas, os, etc.       
![anaconda](https://github.com/zhouchw5/Python_excel.github.io/blob/master/anaconda.png)              
_Figure 1. Spyder is a powerful integrated development environment with advanced editing, interactive testing, debugging and introspection features._          
          
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
From a converse perspective, the coding below would show us how to build up a new xlsx. file and write in our selected data:         
``` python         
test_dataframe = pd.DataFrame(test_01)
wx = Workbook()
wy = wx.create_sheet("test_sheet_output", 0)
for r in dataframe_to_rows(test_dataframe, index = False, header = True):
    wy.append(r)
wx.save(r'D:\Drivers\python_test\test_02.xlsx')      
```           
where we have added a new file named test_02 to our proj_folder, with the selected data from file test_01 writen in. We can also use the xlwt or the xlsxwriter as our operation engine to achieve similar functions.        
             
### Basic Operations with Data         
After building up the bi directional bridge between Python and Excel, some basic operations can be introduced to the process with data exploring in Python before they go back to their unit lattices in xlsx.files.                
            
One of the elemental assignments in supply chain management is the demand/supply match. In this process, three main variables in report are Items, Amount and Date Time. Items would be divided into raw-materials, different levels of semi-products and products. The systematic BOM (bill of materials) would store all these related data and integrate different levels of items in actual analytics cases. For simplicity of the discussion, we can compress the BOM into two levels: parent items and son items, with the collocation ratio between both of which. You can imagine a fictitious black box containing all levels of semi-products and their sophisticated relation. The two objects in the two edges of the black box are parent items and son items. Thus in this article we just take the simplified model to discuss some basic usages of Python to our working scenarios, with no need to open the black box.          
          
Thus in terms of parent items, three main variables (Items, Amount, Date Time) can be simplified as:          
![parentitemsforecast](https://github.com/zhouchw5/Python_excel.github.io/blob/master/parentitemforecast.png)          
_**Figure 2.** In actual cases, in terms of the index 'demand_type', other main tabs than 'forecast' should be 'order', 'supplier_response', 'forecast_gap', etc. 'Forecast' represents the future demand data of an item, which is based on the analysis of the historical outbound data by using some statistics models and the right mix of the major projects information collected by front-line colleagues._                        
Thus a basic technique is to collect the data of items, within the selected index and the time interval under considering. Generally we would consider the time interval lasting 13 weeks (equal to one season/three months) since the starting week.           
``` python      
start_wk = '2018-05-21'           
end_wk = datetime.timedelta(weeks = 13) + datetime.datetime.strptime(start_wk, "%Y-%m-%d")        
end_wk = str(end_wk)[:10]       
```         
We utilize the str function to obtain the alphabetic string form of 'end_wk', the datatime.datetime object, with the length of 10 characters we need. If we use the datetime.datetime.object directly in our subsequent coding, some bugs would come out because the datetime.datetime object has no attribute for some functions like 'startwith' or 'str'.                  
         
``` python         
def read_fcst(proj_folder_path, file_name, sheetname):
    current_file = os.path.join(proj_folder_path, 'test//20180521', file_name)
    fcst = pd.read_excel(current_file, sheet_name = sheetname)
    fcst.columns = fcst.columns.map(str)        
    
    if 'demand_type' in fcst.columns:
        f_type_col = 'demand_type'      
    elif 'Data Measures' in fcst.columns:
        f_type_col = 'Data Measures'
    else:
        raise ValueError("Error: the model cannot find the measure.")
    
    fcst = fcst[fcst[f_type_col] == 'forecast']
    if 'item' in fcst.columns:
        fcst.rename(columns = {'item':'parent_item'}, inplace = True)
    elif 'ITEM' in fcst.columns:
        fcst.rename(columns = {'ITEM':'parent_item'}, inplace = True)
    else:
        raise ValueError("Error: the 'parent_item' does not exist.")
        
    all_timebucket = []
    for col in fcst.columns:
         if col.startswith('20') & (col >= start_wk) & ( col <= end_wk):
               all_timebucket.append(col)        

    all_timebucket.sort()
    keep_cols = all_timebucket[:13]
    keep_cols.append('parent_item')
    fcst = fcst[keep_cols]
    
    demand_fcst = pd.melt(fcst, id_vars = ['parent_item'],  var_name = 'lg_wk', value_name = 'qty')
    demand_fcst['qty'] = demand_fcst['qty'].fillna(0) 
    
    demand_fcst = demand_fcst.groupby(['parent_item', 'lg_wk'], as_index = False)['qty'].sum()
    demand_fcst['lg_wk'] = demand_fcst['lg_wk'].str[:10]  
    return demand_fcst       
    
fcst_df = read_fcst(proj_folder, 'overview.xlsx', 'TYPICAL CONFIGURATION FORECAST')
```        
We define a new function named read_fcst, selecting the forecast data of each parent item within the time bucket from a table like figure 2. in an xlsx. file and melting the data into the columns form with the identification variable 'parent_item', variable name 'lg_wk' and the name of value 'qty', as shown below.           
![columnsformofforecastdata](https://github.com/zhouchw5/Python_excel.github.io/blob/master/columnsformofforecastdata.png)          
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; _Figure 3. The forecast data melted in columns_        
        
        
    



















