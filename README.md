# web-app-2
repo
import streamlit as st
from peewee import SqliteDatabase, Model, CharField
import bcrypt
import pandas as pd
import plotly.express as px
import os
import base64

# Database setup with Peewee
db = SqliteDatabase("user_data.db")

class User(Model):
    email = CharField(unique=True)
    password = CharField()

    class Meta:
        database = db

# Create the database and user table if not exist
db.connect()
db.create_tables([User], safe=True)

# Helper functions for authentication
def hash_password(password):
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(password, hashed_password):
    return bcrypt.checkpw(password.encode(), hashed_password.encode())

def add_user(email, password):
    if User.select().where(User.email == email).exists():
        return False, "User already exists."
    hashed_password = hash_password(password)
    User.create(email=email, password=hashed_password)
    return True, "User created successfully!"

def authenticate_user(email, password):
    user = User.select().where(User.email == email).first()
    if user and verify_password(password, user.password):
        return True, user.email
    return False, None

# Streamlit page configuration
st.set_page_config(page_title="Air Quality Index Visualisation", layout="wide")

# Track the authenticated user
if "authenticated" not in st.session_state:
    st.session_state.authenticated = False
    st.session_state.user_email = None

# Function to encode image in base64
def get_base64_image(image_path):
    with open(image_path, "rb") as img_file:
        return base64.b64encode(img_file.read()).decode()

# Apply background image using custom CSS
def add_background_image(image_file):
    st.markdown(
        f"""
        <style>
        .stApp {{
            background: url(data:image/png;base64,{image_file});
            background-size: cover;
            background-repeat: no-repeat;
            background-attachment: fixed;
            height: 100vh;
            width: 100vw;
            overflow: hidden;
            color: darkbrown;
        }}
        .header {{
            text-align: center;
            font-size: 36px;
            font-weight: bold;
            color: darkbrown;
            margin-top: 20px;
        }}
        .footer {{
            text-align: center;
            color: darkbrown;
            margin-top: 50px;
        }}
        .stTextInput > div > input {{
            font-size: 16px;
            border: 2px solid darkbrown;
            border-radius: 10px;
            padding: 10px;
            background-color: var(--input-background-color);
            color: darkbrown;
        }}
        .stButton > button {{
            background-color: darkbrown;
            color: var(--button-text-color);
            border-radius: 10px;
            padding: 10px;
            font-size: 16px;
            border: none;
        }}
        .stButton > button:hover {{
            background-color: var(--button-hover-color);
        }}
        .stRadio > div > label {{
            font-size: 16px;
            color: darkbrown;
        }}
        .stDataFrame {{
            border: 2px solid darkbrown;
            border-radius: 10px;
        }}
        .stSelectbox > div > div {{
            font-size: 16px;
            border: 2px solid darkbrown;
            border-radius: 10px;
            padding: 10px;
            background-color: var(--input-background-color);
            color: darkbrown;
        }}
        :root {{
            --primary-color: darkbrown; /* Default to dark brown for light mode */
            --secondary-color: #666666; /* Default to dark gray for light mode */
            --button-text-color: #FFFFFF; /* Default to white for dark mode */
            --button-hover-color: #333333; /* Default to dark gray for dark mode */
            --input-background-color: #FFFFFF; /* Default to white for input backgrounds */
        }}
        @media (prefers-color-scheme: dark) {{
            :root {{
                --primary-color: darkbrown; /* Dark brown for dark mode */
                --secondary-color: #AAAAAA; /* Light gray for dark mode */
                --button-text-color: #000000; /* Black for light mode */
                --button-hover-color: #555555; /* Darker gray for dark mode */
                --input-background-color: #333333; /* Dark gray for input backgrounds */
            }}
        }}
        </style>
        """,
        unsafe_allow_html=True
    )

# Set background image
background_image_path = "cccc.jpeg"
if os.path.exists(background_image_path):
    image_base64 = get_base64_image(background_image_path)
    add_background_image(image_base64)
else:
    st.warning("Background image not found.")

# Authentication workflow
def show_auth_page():
    st.markdown('<div class="header">Welcome to Application</div>', unsafe_allow_html=True)
    auth_option = st.radio("Choose an option:", ["Login", "Sign Up"])

    if auth_option == "Login":
        st.subheader("Login")
        email = st.text_input("Email", key="login_email")
        password = st.text_input("Password", type="password", key="login_password")
        if st.button("Login"):
            success, user_email = authenticate_user(email, password)
            if success:
                st.session_state.authenticated = True
                st.session_state.user_email = user_email
                st.success(f"Welcome back, {user_email}!")
                st.experimental_set_query_params(authenticated="true")
        else:
            st.error("Invalid email or password.")
    elif auth_option == "Sign Up":
        st.subheader("Sign Up")
        email = st.text_input("Email", key="signup_email")
        password = st.text_input("Password", type="password", key="signup_password")
        confirm_password = st.text_input("Confirm Password", type="password", key="signup_confirm_password")
        if st.button("Sign Up"):
            if email and password and password == confirm_password:
                success, message = add_user(email, password)
                if success:
                    st.success(message)
                else:
                    st.error(message)
            else:
                st.warning("Please provide matching email and password.")

# Main application workflow
def show_main_app():
    st.sidebar.title("Menu")
    st.sidebar.write(f"Logged in as: {st.session_state.user_email}")
    if st.sidebar.button("Logout"):
        st.session_state.authenticated = False
        st.session_state.user_email = None
        st.experimental_set_query_params(authenticated="false")

    st.title("Air Quality Index Visualisation")
    st.write("Explore Air Quality Index visualisation and enjoy insights! The visualization displays a global map with varying color gradients representing different Air Quality Index (AQI) categories, from 2020 to 2023 Colors range from: Light blue and green, indicating good air quality (AQI 0-50) Yellow and orange, signifying moderate air quality (AQI 51-100) Red and purple, representing poor air quality (AQI 101-200) and hazardous conditions (AQI 201+) The map reveals trends and fluctuations in global air quality over the three-year period, highlighting areas of improvement and decline")

    # Embed URL from Power BI
    embed_url = "https://app.powerbi.com/view?r=eyJrIjoiYzdlMWJmZjEtMjEzMS00ZWNjLTk5Y2UtYWJiZjhjNzhkNjcyIiwidCI6IjVlNDMwMDE5LWZlMmItNDg3Ni04OTk3LTg5MzkxMmE2YjkyYiJ9"
    iframe = f"""
    <iframe 
        src="{embed_url}" 
        width="100%" 
        height="600" 
        frameborder="0" 
        allowFullScreen="true">
    </iframe>
    """
    st.markdown(iframe, unsafe_allow_html=True)

    # Load dataset (example file or user-uploaded)
    dataset_file = "Cleaned_data_2020-23 (1).csv"
    if os.path.exists(dataset_file):
        df = pd.read_csv(dataset_file)
        st.subheader("Dataset of AQI")
        
        # Button to download the dataset
        csv = df.to_csv(index=False).encode('utf-8')
        st.download_button(
            label="Download Dataset as CSV",
            data=csv,
            file_name='Cleaned_data_2020-23.csv',
            mime='text/csv',
        )

        st.dataframe(df)  # Display the full dataset

        st.subheader("Column Distribution")
        col = st.selectbox("Select a column for visualization:", df.columns)
        if df[col].dtype in ["int64", "float64"]:
            fig = px.histogram(df, x=col, title=f"Distribution of {col}")
            st.plotly_chart(fig)
        else:
            st.write("Selected column is not numeric.")
    else:
        st.error(f"Dataset file not found: {dataset_file}")

# Main app flow
query_params = st.query_params
if query_params.get("authenticated") == ["true"]:
    st.session_state.authenticated = True

if not st.session_state.authenticated:
    show_auth_page()
else:
    show_main_app()

st.markdown('<div class="footer">©2025 Air Quality Index Visualisation</div>', unsafe_allow_html=True)
    
