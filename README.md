# E-Commerce-Data-Pipeline-BI-Dashboard

Developed an end-to-end E-Commerce data pipeline integrating Streamlit frontend, Python (VS Code), MySQL database, and Power BI dashboard reporting. Built customer and employee management modules with real-time product, order, and inventory tracking. Designed relational database schema in MySQL and implemented CRUD operations using SQL queries. Connected the database to Power BI to create interactive dashboards for sales analysis, stock monitoring, and business insights.


<img width="1908" height="877" alt="Screenshot 2026-02-17 111700" src="https://github.com/user-attachments/assets/5c4110df-18a8-4bbf-89ba-7a3dd65bf486" />

import streamlit as st

import mysql.connector
import pandas as pd
from tabulate import tabulate
from streamlit_option_menu import option_menu

# Database connection
import mysql.connector

con = mysql.connector.connect(
    host="localhost",
    user="root",
    password="mani12",
    database="retail_store"
)


osm = con.cursor()

def get_product_id():
    cus_id_lst=[]
    cus_id_qry="select product_id from products"
    osm.execute(cus_id_qry)
    data1=osm.fetchall()
    for i in data1:
        cus_id_lst.append(i[0])
    return cus_id_lst

def get_product_name():
    cus_user_name_lst=[]
    qry="select product_name from products"
    osm.execute(qry)
    data2=osm.fetchall()
    for i in data2:
        cus_user_name_lst.append(i[0])
    return cus_user_name_lst

def get_user_password():
    cus_user_pwd_lst=[]
    qry="select user_password from customer"
    osm.execute(qry)
    data3=osm.fetchall()
    for i in data3:
        cus_user_pwd_lst.append(i[0])
    return cus_user_pwd_lst

def stock_updates(product_id,quantity):
    qry="update products set quantity=%s where product_id=%s"
    val4=(quantity,product_id)
    osm.execute(qry,val4)
    con.commit()  


def customer_signup():
        cname,do,gen=st.columns(3)
        cus_name = cname.text_input("Name",placeholder="Enter Your Name")
        dob = do.text_input("Date Of Birth",placeholder="Date of Birth")
        gender=gen.selectbox("Gender",["","Male","Female","TransGender"],placeholder="Enter Your Gender")

        mail,ph=st.columns(2)
        email=mail.text_input("E-Mail",placeholder="Enter the Email")
        phone=ph.text_input("Phone Number",placeholder="Enter Your Phone Number")
        
        cit,stat=st.columns(2)
        city=cit.text_input("City",placeholder="Enter Your City")
        state=stat.text_input("State",placeholder="Enter Your State")

        usr,pwd,pwd1=st.columns(3)
        user_name=usr.text_input("UserName",placeholder="Enter the UserName")
        user_password=pwd.text_input("Password",placeholder="Enter the Password",type="password")
        user_password_reenter=pwd1.text_input("Re-enter PassWord",placeholder="Enter the Re-enter Password",type="password")

        check = st.checkbox("I Agreed")
        button=st.button("Sign Up")

        if button:
            if not cus_name:
                st.error("Fill your name")
            
            if user_password == user_password_reenter:
                qry = "insert into customer(customer_name,dob,gender,email,phone_no,city,state,user_name,user_password)values(%s,%s,%s,%s,%s,%s,%s,%s,%s)"
                val1=(cus_name,dob,gender,email,phone,city,state,user_name,user_password)
                osm.execute(qry,val1)
                con.commit()
                st.success("Registeration Is Done")
                st.balloons()
            else:
                st.error("Re-Enter the Password")

def get_order_id():
    qry="select order_id from orders"
    osm.execute(qry)
    result = osm.fetchall()
    list_of_order = []
    for i in result:
        list_of_order.append(i[0])
    return list_of_order

def get_customer():
    qry = "select customer_id from customer"
    osm.execute(qry)
    result = osm.fetchall()
    list_of_ids = []
    for i in result:
        list_of_ids.append(i[0])
    return list_of_ids

def get_product_name(product_name):
    qry = "select product_id from products where product_name = %s"
    osm.execute(qry, (product_name,))
    data1 = osm.fetchall()
    if data1 and len(data1[0]) > 0:
        return data1[0][0]
    else:
        return None

def product_stock(product_id):
    qry="select quantity from products where product_id=%s"
    osm.execute(qry,(product_id,))
    data5=osm.fetchall()
    if data5 and len(data5[0]) > 0:
        return data5[0][0]
    else:
        return None

def new_order(customer_id, product_id, product_name, quantity, price_res, total_amount):
    qry = """
        insert into orders
        (customer_id, product_id, product_name, quantity, price_per_quantity, total_amount)
        values
        (%s, %s, %s, %s, %s, %s)
    """
    osm.execute(qry,(customer_id, product_id, product_name, quantity, price_res, total_amount))
    con.commit()


def cancel_order(order_id):
    qry="delete from orders where order_id=%s"
    osm.execute(qry,(order_id,))
    con.commit()


def get_product_id(order_id):
    qry = "select product_id from orders where order_id=%s"
    val1 = (order_id,)
    osm.execute(qry, val1)  
    dat8 = osm.fetchall()
    if dat8 and len(dat8[0]) > 0:
        return dat8[0][0]
    else:
        return None

def get_order_qty(order_id):
    qry="select quantity from orders where order_id=%s"
    osm.execute(qry,(order_id,))
    data9=osm.fetchall()
    if data9 and len(data9[0]) > 0: 
        return data9[0][0] 
    else:
        return None


# Define your Streamlit app
def customer_login():
    st.title("**Customer Login Page**")
    customer_id = st.number_input("Enter the Customer Id",step=1)
    user_name = st.text_input("User Name(If you enter correct username the error will not show)",placeholder="Enter the user name")
    user_password=st.text_input("User Password (If you enter correct password the error will not show)",placeholder="Enter the user password",type="password")
    
    cust_list = get_customer()
    cust_username=get_user_name()
    cust_pwd=get_user_password()

    if customer_id in cust_list:
        if user_name in cust_username:
            if user_password in cust_pwd:
                st.success("Login successful!")
                st.markdown("**Select an option**")
                user = st.selectbox("", ["None","View Orders", "Book New Orders", "Cancel Order","Details Update"])

                if user == "View Orders":
                    qry = "select * from orders where customer_id = %s"
                    val = (customer_id,)
                    osm.execute(qry, val)
                    result = osm.fetchall()
                    table_headings = [desc[0] for desc in osm.description]
                    # Create a list that includes the headings as the first row
                    table_data = [table_headings] + result
                    st.table(table_data)

                elif user == "Book New Orders":
                    qry = "select * from products"
                    osm.execute(qry)
                    result = osm.fetchall()
                    mm = pd.DataFrame(result,columns=["product_id","product_name","quantity","price_per_quantity"])
                    st.table(mm)

                    product_name = st.text_input("**Enter the Product Name**")
                    quantity = st.number_input("**Enter the quantity**",value=0)

                    product_id = get_product_name(product_name)
                    current_stock = product_stock(product_id)
            
                    qry = "select price_per_quantity from products where product_name = %s"
                    osm.execute(qry, (product_name,))
                    price_res = osm.fetchall()
                    
                    if price_res:
                        price_res = price_res[0][0]
                    else:
                        print("price_res is empty or has an unexpected format.")
                    
                    total_amount = quantity * price_res
            
                    if st.button("Submit"):
                        if current_stock == 0:
                            st.warning("Out Of Stock")

                        elif current_stock>=quantity:
                            new_order(customer_id, product_id, product_name, quantity, price_res, total_amount)
                            change_product = current_stock - quantity
                            stock_updates(product_id,change_product)
                            st.success("**Order placed successfully**")
                        else:
                            st.error("**Something went wrong**")

                elif user == "Cancel Order":
            
                    qry = "select * from orders where customer_id = %s"
                    val = (customer_id,)
                    osm.execute(qry, val)
                    result = osm.fetchall()
                    men = pd.DataFrame(result,columns=["order_id","customer_id","product_id","product_name","quantity","price_per_quantity","total_amount"])
                    st.table(men)

                    cust_ord_list = get_order_id()
                    order_id = st.number_input("Enter the Order Id",step=1)
                    product_id = get_product_id(order_id)
                    current_stock = product_stock(product_id)
                    quan = get_order_qty(order_id)


                    if st.button("Submit"):

                        if order_id in cust_ord_list:
                            cancel_order(order_id)
                            change_product = current_stock + quan
                            stock_updates(product_id,change_product)

                            st.success("**Order Cancelled**")
                            
                elif user == "Details Update":
                    st.markdown("Customer Update Details")
                    ct_update=st.selectbox("",["","Personal Details","UserName and Password"])
                    if ct_update=="Personal Details":
                        personal_details=st.selectbox("",["Name Update","Date of Birth","Gender","Email","Phone Number","City","State"])
                        if personal_details == "Name Update":
                            name=st.text_input("Name",placeholder="Enter to Change Your Name")
                            if st.button("Submit"):
                                qry="update customer set customer_name=%s where id=%s"
                                value =(name,customer_id)
                                osm.execute(qry,value)
                                con.commit()
                                st.success("Your Name Update is Change")

                        elif personal_details == "Date of Birth":
                            dob=st.text_input("Date Of Birth (DD/MM/YYYY)",placeholder="Enter the Change to DOB")

                            if st.button("Submit"):
                                qry="update customer set dob=%s where customer_id=%s"
                                osm.execute(qry,(dob,customer_id))
                                con.commit()
                                st.success("Your Date of Birth Update is Change")

                        elif personal_details == "Gender":
                            gen=st.selectbox("",["","Male","Female"])

                            if st.button("Submit"):
                                qry="update customer set gender=%s where customer_id=%s"
                                osm.execute(qry,(gen,customer_id))
                                con.commit()
                                st.success("Your Gender Update is Change")

                        elif personal_details == "Email":
                            mail=st.text_input("Email",placeholder="Enter to Change the Email")

                            if st.button("Submit"):
                                qry="update customer set email=%s where customer_id=%s"
                                osm.execute(qry,(mail,customer_id))
                                con.commit()
                                st.success("Your Email Update is Change")

                        elif personal_details == "Phone Number":
                            pn=st.text_input("Phone Number(12345 67890)",placeholder="Enter the Change to Phone Number")

                            if st.button("Submit"):
                                qry="update customer set phone_no=%s where customer_id=%s"
                                osm.execute(qry,(pn,customer_id))
                                con.commit()
                                st.success("Your Phone Number Update is Change")

                        elif personal_details == "City":
                            cit=st.text_input("City",placeholder="Enter the Change to City Name")

                            if st.button("Submit"):
                                qry="update customer set city=%s where customer_id=%s"
                                osm.execute(qry,(cit,customer_id))
                                con.commit()
                                st.success("Your City Update is Change")

                        elif personal_details == "State":
                            state=st.text_input("State",placeholder="Enter the Change to State Name")

                            if st.button("Submit"):
                                qry="update customer set state=%s where customer_id=%s"
                                osm.execute(qry,(state,customer_id))
                                con.commit()
            
                                st.success("Your State Name Update is Change")
                    elif ct_update=="UserName and Password":
                        up=st.selectbox("",["User Name","Password"])
                        if up=="User Name":
                            new_username=st.text_input("User Name",placeholder="Enter the Change to User Name")

                            if st.button("Submit"):
                                qry="update customer set user_name=%s where customer_id=%s"
                                osm.execute(qry,(new_username,customer_id))
                                con.commit()
            
                                st.success("Your User Name Update is Change")
                            
                        elif up=="Password":
                            new_password=st.text_input("Password",placeholder="Enter the Change Paasword")

                            if st.button("Submit"):
                                qry="update customer set user_password=%s where customer_id=%s"
                                osm.execute(qry,(new_password,customer_id))
                                con.commit()
        
                                st.success("Your Password Update is Change")
            else:
                st.error("Wrong Password")
        else:
            st.error("Wrong Username")            
    else:
        st.write("***In This Page You Need to Enter the User Name And User Password***")

def employee_signup():

    name,pwd=st.columns(2)
    first_name=name.text_input("First Name (Captial Letter):",placeholder="Enter Your Full Name")
    password_input=pwd.text_input("Password (ABC123@abc)",placeholder="Enter The Password",type="password")

    repwd,no=st.columns(2)
    password_input_reenter=repwd.text_input("Re-Enter Password",placeholder="Enter The Re-enter Password",type="password")
    ph_no=no.text_input("Phone Number",placeholder="12345 67890")

    if st.button("Sign Up"):
        if password_input==password_input_reenter:

            qry="insert into employee(first_name,password_input,ph_no)values(%s,%s,%s)"
            val=(first_name,password_input,ph_no)
            osm.execute(qry,val)
            con.commit()

            st.success("Succesfully Register")
            st.balloons()


def emp_get_username():
    get_user_name_lst=[]
    qry="select first_name from employee"
    osm.execute(qry)
    data10=osm.fetchall()
    for i in data10:
        get_user_name_lst.append(i[0])
    return get_user_name_lst

def emp_get_password():
    get_password_lst=[]
    qry="select password_input from employee"
    osm.execute(qry)
    data11=osm.fetchall()
    for i in data11:
        get_password_lst.append(i[0])
    return get_password_lst

def product_entry(product_name,quantity,price_per_quantity):
    qry="insert into products(product_name,quantity,price_per_quantity)values(%s,%s,%s)"
    val=(product_name,quantity,price_per_quantity)
    osm.execute(qry,val)
    con.commit()

def product_quan_update(product_id,quantity):
    qry="update products set quantity=%s where product_id=%s"
    val=(quantity,product_id)
    osm.execute(qry,val)
    con.commit()

def product_name_update(product_id,product_name):
    qry="update products set product_name=%s where product_id=%s"
    val=(product_name,product_id)
    osm.execute(qry,val)
    con.commit()

def product_ppq(product_id,ppq):
    qry="update products set price_per_quantity=%s where product_id=%s"
    val=(ppq,product_id)
    osm.execute(qry,val)
    con.commit()

def get_productid_tab(product_name):
    qry = "select product_id from products where product_name=%s"
    val1 = (product_name,)
    osm.execute(qry, val1)  
    dat8 = osm.fetchall()
    if not dat8:  # Check if the result is empty
         ValueError(f"No product found for order_id {product_name}")
    return dat8[0][0]

def employee_login():

    employee_name=st.text_input("Enter Your Name",placeholder="Enter the Your Name")
    employee_password=st.text_input("Enter Password",placeholder="Enter Your Password",type="password")

    emp_user_name_in=emp_get_username()
    emp_user_pwd_in=emp_get_password()

    if employee_name in emp_user_name_in:
        if employee_password in emp_user_pwd_in:

            st.success("Successfully Login")
            st.title("Options")
            
            emp_admin=st.selectbox("",["Product Entry","Product Quantity Update","Employee Details Update"])

            qry="select*from products"
            osm.execute(qry)
            data11=osm.fetchall()
            table_heading=[desc[0] for desc in osm.description]
            table_data=[table_heading] + data11
            st.table(table_data)


            if emp_admin == "Product Entry":
                product_name=st.text_input("Product Name",placeholder="Enter the New Product Name ")
                quantity=st.number_input("Quantity",step=1,value=0,placeholder="Enter the Quantity You need")
                price_per_quantity=st.number_input("Price Per Quantity",step=1,value=1,placeholder="Enter the Quantity You need")

                if st.button("Submit"):

                    product_entry(product_name,quantity,price_per_quantity)

                    st.success("Product Has inserted")
        
            elif emp_admin == "Product Quantity Update":
                st.markdown("Select Option To Update")
                choice=st.selectbox("",["Product Name","Quantity","Price Per Quantity"])
                if choice =="Product Name":
                    product_id=st.text_input("Enter the Product Id")
                    ch_name=st.text_input("Enter the Product Name")

                    if st.button("Update"):
                        product_name_update(product_id,ch_name)
                        st.success("Name Change is Updated")
                
                    elif choice == "Quantity":
                        product_name=st.text_input("Enter the Product Name")
                        quantity=st.number_input("Enter the Quantity",step=1,value=0)

                        product_id=get_productid_tab(product_name)

                        if st.button("Update"):
                            curr_stock=product_stock(product_id)

                            ch=curr_stock+quantity

                            stock_updates(product_id,ch)

                            st.success("Quantity Change is update")

            elif choice == "Price Per Quantity":

                product_id=st.number_input("Enter the Product Id",step=0,value=1)
                ppq=st.number_input("Enter the Price Per Quantity",step=0,value=1)

                if st.button("Update"):
                    product_quan_update(product_id,ppq)
                    st.success("Price Change is Update")
        else:
            st.error("Wrong Password")            
    else:
        st.write("***Enter Correct Name and Password then only the error will not show***")        



cus_option=""
emp_option=""



#python frontend
st.sidebar.title("E-Commerce Application Online Store")

with st.sidebar:
    selected = option_menu("Main Menu",["Home Page","Customer","Employee"])
    if selected == "home page":
        st.markdown(
            '''
            <div style="text-align:center;">
                <h1>Welcome to Our E-Commerce Store</h1>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.markdown(
            '''
            <div style="text-align:center;">
                <h4>Discover a world of convenience and endless possibilities at our online store. Shop now and experience the future of retail!</h4>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.image("retail.mp4", width=700)
    if selected == "Customer":
        st.sidebar.title("Customer")
        cus_option = option_menu("", ["Signup", "Login"])
        
    elif selected == "Employee":
        st.sidebar.title("Employee")
        emp_option = option_menu("", ["Signup", "Login"])


# new.py
    
# ---------------- CUSTOMER ----------------
if selected == "Customer":

    if cus_option == "Signup":
        customer_signup()

    elif cus_option == "Login":
        st.markdown(
            '''
            <div style="text-align:center;">
                <h4>Sign in To Unlock Exclusive Deals and Offers</h4>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.markdown(
            '''
            <div style="text-align:center;">
                <h4>Every great journey starts with a single step. Login to begin yours</h4>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.image("customer_login.jpg", width=500)
        customer_login()


# ---------------- EMPLOYEE ----------------
elif selected == "Employee":

    if emp_option == "Signup":
        st.markdown(
            '''
            <div style="text-align: center;">
                <h3>Employee Registration Page</h3>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.markdown(
            '''
            <div style="text-align: center;">
                <h4>Join our team and help shape the future of E-Commerce innovation!</h4>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.image("employee_sign_up.jpg", width=600)
        employee_signup()

    elif emp_option == "Login":
        st.markdown(
            '''
            <div style="text-align:center;">
                <h3>Employee Login Page</h3>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.markdown("**Your Efforts Are Truly Appreciated**")

        st.markdown(
            '''
            <div style="text-align:center;">
                <h4>Welcome to Employee Login Page</h4>
            </div>
            ''',
            unsafe_allow_html=True
        )

        st.image("staff_login_form.jpg", width=700)
        employee_login()

if emp_option=="Signup":
    st.markdown('''
    <div style="text-align: center;">
    <h3>Employee Registration Page</h3>
    </div>''',
     unsafe_allow_html=True
    )
    

    st.markdown('''
    <div style="text-align: center;">
    <h4>Join our team and help shape the future of E-Commerce innovation!</h4>
    </div>''',
     unsafe_allow_html=True
    )

    st.image("employee_sign_up.jpg",width=600)

    st.markdown('''
    <div style="text-align: center;">
    <h5>Employee Can Work In This Process</h5>
    </div>''',
     unsafe_allow_html=True
    )

    employee_signup()

elif emp_option=="Login":
    st.markdown(
            '''<div style="text-align:center;"
            <h3>Employee Login Page</h3>
            </div>''',
            unsafe_allow_html=True
        )

    st.markdown("Your Efforts Are Truly Appreciated")

    st.markdown(
            '''<div style="text-align:center;"
            <h4>Welcome to Employee Login Page</h4>
            </div>''',
            unsafe_allow_html=True
        )
    
    st.image("staff_login_form.jpg",width=700)
    employee_login()
