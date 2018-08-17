# Some Basic Usages of Python in My Daily Work_(II)         
        
        
- **Still with my original insistence, your footprint in my website would not be marked or stored. I always wish a zero intersection with me after your reading, nothing but some interesting memory merely in your mind, no storage for the Like or reader amount. Your ever presence is enough.**           

In the letter _Some Basic Usages of Python in My Daily Work (I)_, we have known how Python could exchange information with an xlsx. file and some preliminary operations to edit data performed in Python. Based on these preliminary operations and some extended coding knowledge shown in this article, I would develop a simple and skeleton model to perform the decision-making process and obtain an elementary demand/supply match report.        
            
Yours,            
Zhou Mr.Chuwei            
2018.08.10

------------------------------           
           
## Our Working Schedule          
**_Working Network_**           
Our working calendar had three lines: the planning line, the ordering line and the implementing line. The planning line and the ordering line are the activating source of the whole operating process. Generally, dots linked up in planning line are the so-called planning objects. Dots linked up in sales ordering line are the Quotation Items. In actual case, planning objects would be determined in different levels of semi-products or raw materials. Conversely, the implementing line is based on the supply data of raw materials. Activated by the supply capability of raw materials, manufacturing processes of different levels of semi-products would come up with the final implementary sales plan of Quotation Items and related products.                 
         
                  
**_Working Pattern_**          
Different patterns of resource planning depend on the weights of forecast data or order data to activate the operations as the source of demand, like the pull pattern 100% from forecast data and the push pattern 100% from order data. Though in actual cases we are often oriented by the outcomes from the hedging between forecast and order, with the right mix of the pull and push patterns, I only discuss the pull pattern in this article with the activation from forecast data of parent items which we have red via Python in the (I) edition. And simply these parent items would also acts as the planning objects in our description here.           
        
        
**_Bridge Connecting the Parent Items And Son Items_**            
In the edition (I), we have imagined a black box containing all levels of semi-products to compress the 'medium' between parent items and son items. The black box is the simplified introduction in positive-direction with the forecast information of parent items (planning objects) to son items (raw materials). As we have mentioned in _Working Network_, besides the activation in positive-direction from the forecast data of parent items, We also need a reversed process from the supply information of son items to develop the implementing plan of Quotation Items, which is a sophisticated procedure considering the supply of materials and the manufacturing progress of all levels of semi-products. Again for simplicity in this letter, we should imagine a white box compressing all this complexity into a basic model using the Cannikin Law, which would determine the a parent item's supply schedule based on the supply capability of all its son items. Before using the Cannikin Law, the white box has also built another path of the opposite direction,  determining the weights of a son item's supply to transfer to all its parent items. After the white box finishes its bi directional assignments, we can obtain the final implementing sales plan of Quotation Items and achieve a close loop in our working network.        
          
          
## Connecting the Planning Line of Parent Items to the Supply of Son Items        
As the activating engine of our working schedule, the planning line and the ordering line are the estimated labels in front line like the navigation lighthouse. The navigation lighthouse has been lighted up in last letter (_Some basic usages of Python in My Daily Work (I)_) where we have introduced how to read the forecast data of parent items via Python. Then we would introduce the simplified BOM (bill of materials) to translate the forecast data of parent items into the demand of son items.               
![BOM](https://github.com/zhouchw5/Python_excel.github.io/blob/Python/BOM.jpg)                
**Table 1.** _Here's the bridge connecting parent items and son items, where the column 'P_QTY' represents the ratio of a parent item to a son item. For example, one parent item 02311VDU would be configured with 12 son items labelled in 06210444._               
           
We can define a function to read the connection shown above:            
``` python     
def read_p2s_ratio(proj_folder_path, file_name):     
    p2s_file = os.path.join(proj_folder_path, 'data\data_model', file_name)    
    df = pd.read_excel(p2s_file, sheet_name = 'BOM')
    df.columns = map(str.lower, df.columns)      
    df.rename(columns = {'SON_ITEM':'son_item', 'PARENT_ITEM':'parent_item', 'P_QTY':'p_qty'}, inplace = True)
    p2s_dim_df = df[['parent_item', 'son_item', 'p_qty']]
    return p2s_dim_df   
```       
          
With the ratio data between parent items and son items, we can define a function to integrate the total demand of a son item from all its parent items' data.            
``` python      
def fcst_no06_2sitem_df(fcst_no_06_df, p2s_dim_df):       
    fcst_parent_ratio_son = fcst_no_06_df.merge(p2s_dim_df, on = ['parent_item'], how = 'left')
    fcst_parent_ratio_son = fcst_parent_ratio_son[fcst_parent_ratio_son['son_item'].notnull()]
    exception_pitem_set = set(fcst_parent_ratio_son[~fcst_parent_ratio_son['son_item'].notnull()]['parent_item']) 
    if (len(exception_pitem_set))>0:
       fcst_parent_ratio_son['s_qty'] = fcst_parent_ratio_son ['qty']* fcst_parent_ratio_son ['p_qty']
       fcst_parent_ratio_son = fcst_parent_ratio_son[['son_item', 'parent_item', 'lg_wk', 's_qty']]
    return fcst_parent_ratio_son   
```
         
Then based on the demand data converged to each son item, we would come to some puzzles like: can the supply of a son item fulfill all the demand? How to arrange the supply schedule to control the inventory if satisfying the demand? How to manage the allocations and achieve the best level of fulfillment if son items' supply is not enough? Thus the bridge extended from the demand of parent items above still retain another half to be completed. This half should be started from the supply of son items.        
            
A function similar to the _read_fcst_ can be defined to grab the supply data:       

``` python        
def read_in_supply(proj_folder_path, file_name, sheetname):    
    current_file = os.path.join(proj_folder_path, 'test\\20180521', file_name)
    df = pd.read_excel(current_file, sheet_name = sheetname)
    df.columns = df.columns.map(str)       
 
    if 'demand_type'in df.columns:
       s_type_col = 'demand_type'
    elif 'Data Measures' in df.columns:
       s_type_col = 'Data Measures'    
    else:
       raise ValueError("Error: the model cannot find the measure.")
    
    df = df[df[s_type_col]=='supplier_response']
    if 'Customer Item' in df.columns:
        df.rename(columns={'Customer Item':'son_item'}, inplace = True)
    elif 'SON_ITEM' in df.columns:
        df.rename(columns={'SON_ITEM':'son_item'}, inplace = True)
    else:
        raise ValueError("Error: the 'son_item' does not exist.")

    all_timebucket = []
    for col in df.columns:
        if col.startswith('20')&(col>=start_wk)&(col<=end_wk):
               all_timebucket.append(col)

    all_timebucket.sort()
    keep_cols = all_timebucket[:13]
    keep_cols.append('son_item')
    df = df[keep_cols]
    
    supply_df = pd.melt(df, id_vars = ['son_item'], var_name = 'lg_wk', value_name = 's_amount')
    supply_df['s_amount'] = supply_df['s_amount'].fillna(0)

    supply_df = supply_df.groupby(['son_item', 'lg_wk'], as_index = False)['s_amount'].sum()
    supply_df['lg_wk'] = supply_df['lg_wk'].str[:10]
    return supply_df
```
       
Simply we just consider the supply data of two son items, a fictitious sample of supply data can be shown as below:          
![supply of son items](https://github.com/zhouchw5/Python_excel.github.io/blob/Python/supply%20of%20son%20items.png) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;**Table 2.**_The supply data of son items_          

Similar to the forecast data in the opposite edge of our working network, the supply data would also be melt into columns form by the function above for subsequent computing.        
           
An intersection point between the foracast and supply has gradually rised to the surface, which should be one of the white box's six surfaces. As we have mentioned previously, we would skip over all the complexity in the white box and choose a simple and feasible way to manage the allocations of son items' supply. Before coding we would firstly show a table describing the allocations process. Actually in some typical projects, parent items are complete machines like servers and storages. And son items are some critical components of complete machines like the HDD (Hard Disk Driver), CPU, SSD (Solid State Driver) and memory. In this letter we just take HDD and memory for examples illustration.                
![allocations management](https://github.com/zhouchw5/Python_excel.github.io/blob/Python/allocations%20management.png)        
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;**Table 3.**_The allocations management_            
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _The ratios are merged from table 1._      
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _The quantity in F columns is the forecast data of parent items we have red in **Some basic Usages of Python in My Daily Work (I)**._            
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _The s_qty in G columns can be obtained by multiplying the D and F columns._        
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _And one sum_qty in H columns can be obtained by summing up the s_qty related to the same son item, like H2 = H3 = H4 = 3396 = G2+G3+G4._          
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _And the rate in I column, determining the weights of supply source allocated to different parent items, is the quotient when sum_qty in H column is divided by s_qty in G column._          
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _The s_amount in J column is the supply data of the son_item from Table 2._             
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _The sub_atp in K column is the product when J column is multiplied by I column.        
<a href="https://www.codecogs.com/eqnedit.php?latex=$&space;\clubsuit&space;$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$&space;\clubsuit&space;$" title="$ \clubsuit $" /></a>. _The AI_qty in L column is the quotient when the sub_atp in K column is divided by the p_qty in D column. Thus the AI_qty is the available supply data of a parent item and we have taken the accuracy to the integral part._          
        
        




       
       
          
               
            

. 

## Allocations Management of Son Items         

      
## Implementing Supply Plan of Parent Items and Demand/Supply Match of Son Items           

       
       
          
              
## Summary         
This letter is not merely about some basic usages of Python. I wish to have the aid of the Python programming to perform my ever working schedule in actual scenarios, without which programming tools are no more than themselves.          

         
         
Best Regards!          
Zhou Mr.Chuwei         
2018.08.10           


    



















