Key Features

Sprint Board Management

Create boards for each sprint with name and date range
Navigate between different sprint boards


Feedback Collection

Team members can submit feedback with their name and feedback type
Categories include "What went well", "What could be improved", and "Action items"


Sentiment Analysis

Each feedback item is analyzed using TextBlob to determine sentiment polarity
Feedback is categorized as Positive, Neutral, or Negative
Visual indicators (color-coding) show sentiment at a glance


Dashboard & Analytics

Overall sentiment score for the sprint
Breakdown of sentiment categories
Sentiment analysis by feedback type
Highlights of most positive and negative feedback
Custom suggestions based on overall sentiment


Export Functionality

Export all feedback and sentiment data as CSV for further analysis



Technical Implementation

Streamlit: For the interactive web interface
Pandas: For data handling and analysis
TextBlob: For natural language processing and sentiment analysis
UUID: For generating unique IDs for boards and feedback items

Running the Application

Make sure you have the required libraries installed:

pip install streamlit pandas textblob

Save the code to a file (e.g., retro_app.py)
Run the application:

streamlit run retro_app.py






Sentiment Analysis App
import streamlit as st
import pandas as pd
from textblob import TextBlob
import datetime
import csv
import io
import uuid

# Set page configuration
st.set_page_config(
    page_title="Agile Retrospective Sentiment Analyzer",
    page_icon="📊",
    layout="wide"
)

# Initialize session state variables if they don't exist
if 'boards' not in st.session_state:
    st.session_state.boards = {}  # Dictionary to store all boards

if 'current_board' not in st.session_state:
    st.session_state.current_board = None

# Function to analyze sentiment
def analyze_sentiment(text):
    analysis = TextBlob(text)
    return analysis.sentiment.polarity

# Function to get sentiment category
def get_sentiment_category(polarity):
    if polarity > 0.05:
        return "Positive"
    elif polarity < -0.05:
        return "Negative"
    else:
        return "Neutral"

# Function to get color based on sentiment
def get_sentiment_color(category):
    if category == "Positive":
        return "green"
    elif category == "Negative":
        return "red"
    else:
        return "gray"

# Function to get suggestion based on overall sentiment
def get_suggestion(sentiment_category):
    if sentiment_category == "Positive":
        return "Keep up the great work! Consider what specific practices contributed to this positive sprint."
    elif sentiment_category == "Negative":
        return "Focus on addressing key blockers in the next sprint. Consider holding a focused problem-solving session."
    else:
        return "The team seems balanced. Check if there are specific areas that could use improvement while maintaining what's working."

# Function to create a new board
def create_board(sprint_name, start_date, end_date):
    board_id = str(uuid.uuid4())
    st.session_state.boards[board_id] = {
        'id': board_id,
        'sprint_name': sprint_name,
        'start_date': start_date,
        'end_date': end_date,
        'feedback': []
    }
    return board_id

# Function to add feedback to a board
def add_feedback(board_id, team_member, feedback_text, feedback_type):
    sentiment_score = analyze_sentiment(feedback_text)
    sentiment_category = get_sentiment_category(sentiment_score)
    
    feedback_id = str(uuid.uuid4())
    feedback_item = {
        'id': feedback_id,
        'team_member': team_member,
        'text': feedback_text,
        'type': feedback_type,
        'sentiment_score': sentiment_score,
        'sentiment_category': sentiment_category,
        'timestamp': datetime.datetime.now()
    }
    
    st.session_state.boards[board_id]['feedback'].append(feedback_item)
    return feedback_item

# Function to export feedback as CSV
def export_as_csv(board_id):
    board = st.session_state.boards[board_id]
    
    # Create a DataFrame from the feedback
    if board['feedback']:
        df = pd.DataFrame(board['feedback'])
        # Exclude the id and convert timestamp to string
        df = df[['team_member', 'text', 'type', 'sentiment_score', 'sentiment_category', 'timestamp']]
        df['timestamp'] = df['timestamp'].astype(str)
        
        # Create a buffer to store the CSV
        csv_buffer = io.StringIO()
        df.to_csv(csv_buffer, index=False)
        
        return csv_buffer.getvalue()
    else:
        return None

# Main app title
st.title("📊 Agile Retrospective Sentiment Analyzer")

# Sidebar for navigation and board creation
with st.sidebar:
    st.header("Navigation")
    
    # Board creation section
    st.subheader("Create New Retrospective Board")
    sprint_name = st.text_input("Sprint Name", placeholder="e.g., Sprint 42")
    col1, col2 = st.columns(2)
    with col1:
        start_date = st.date_input("Start Date", datetime.date.today() - datetime.timedelta(days=14))
    with col2:
        end_date = st.date_input("End Date", datetime.date.today())
    
    if st.button("Create New Board"):
        if sprint_name:
            board_id = create_board(sprint_name, start_date, end_date)
            st.session_state.current_board = board_id
            st.success(f"Board for {sprint_name} created successfully!")
        else:
            st.error("Please enter a sprint name")
    
    # Board selection
    st.subheader("Select Existing Board")
    if st.session_state.boards:
        board_options = {board['sprint_name']: board_id for board_id, board in st.session_state.boards.items()}
        selected_board_name = st.selectbox(
            "Choose a board",
            options=list(board_options.keys()),
            index=0 if st.session_state.current_board is None else list(board_options.values()).index(st.session_state.current_board)
        )
        if selected_board_name:
            st.session_state.current_board = board_options[selected_board_name]
    else:
        st.info("No boards yet. Create a new one!")

# Main content area
if st.session_state.current_board is not None:
    current_board = st.session_state.boards[st.session_state.current_board]
    
    # Board header information
    st.header(f"Sprint: {current_board['sprint_name']}")
    st.write(f"Duration: {current_board['start_date']} to {current_board['end_date']}")
    
    # Tabs for different sections
    tab1, tab2, tab3 = st.tabs(["Add Feedback", "View All Feedback", "Sentiment Analysis"])
    
    # Tab 1: Add feedback form
    with tab1:
        st.subheader("Add New Feedback")
        
        # Form for adding feedback
        with st.form("feedback_form", clear_on_submit=True):
            team_member = st.text_input("Team Member Name", placeholder="Your name")
            feedback_text = st.text_area("Feedback", placeholder="Enter your feedback here...")
            feedback_type = st.selectbox(
                "Feedback Type",
                options=["What went well", "What could be improved", "Action item"]
            )
            
            submit_button = st.form_submit_button("Submit Feedback")
            
            if submit_button:
                if team_member and feedback_text:
                    add_feedback(st.session_state.current_board, team_member, feedback_text, feedback_type)
                    st.success("Feedback submitted successfully!")
                else:
                    st.error("Please fill in all fields")
    
    # Tab 2: View all feedback
    with tab2:
        st.subheader("All Feedback for this Sprint")
        
        if current_board['feedback']:
            # Group by feedback type
            feedback_types = ["What went well", "What could be improved", "Action item"]
            
            for feedback_type in feedback_types:
                filtered_feedback = [f for f in current_board['feedback'] if f['type'] == feedback_type]
                
                if filtered_feedback:
                    st.write(f"### {feedback_type}")
                    
                    for feedback in filtered_feedback:
                        sentiment_category = feedback['sentiment_category']
                        sentiment_color = get_sentiment_color(sentiment_category)
                        
                        with st.container():
                            st.markdown(f"""
                            <div style="border-left: 5px solid {sentiment_color}; padding-left: 10px; margin-bottom: 10px;">
                                <p><strong>{feedback['team_member']}</strong> - <span style="color: {sentiment_color};">{sentiment_category}</span> ({feedback['sentiment_score']:.2f})</p>
                                <p>{feedback['text']}</p>
                                <p><small>{feedback['timestamp'].strftime('%Y-%m-%d %H:%M')}</small></p>
                            </div>
                            """, unsafe_allow_html=True)
        else:
            st.info("No feedback added yet for this sprint.")
        
        # Export functionality
        if current_board['feedback']:
            csv_data = export_as_csv(st.session_state.current_board)
            st.download_button(
                label="Export as CSV",
                data=csv_data,
                file_name=f"{current_board['sprint_name']}_feedback.csv",
                mime="text/csv"
            )
    
    # Tab 3: Sentiment Analysis
    with tab3:
        st.subheader("Sentiment Analysis")
        
        if current_board['feedback']:
            # Convert to DataFrame for easier analysis
            df = pd.DataFrame(current_board['feedback'])
            
            # Calculate overall sentiment
            avg_sentiment = df['sentiment_score'].mean()
            overall_category = get_sentiment_category(avg_sentiment)
            sentiment_color = get_sentiment_color(overall_category)
            suggestion = get_suggestion(overall_category)
            
            # Display overall sentiment
            col1, col2 = st.columns([1, 3])
            with col1:
                st.metric(
                    label="Overall Sentiment Score", 
                    value=f"{avg_sentiment:.2f}", 
                    delta=None
                )
            with col2:
                st.markdown(f"""
                <div style="background-color: {sentiment_color}; color: white; padding: 10px; border-radius: 5px;">
                    <h3 style="margin: 0;">Overall Sentiment: {overall_category}</h3>
                </div>
                """, unsafe_allow_html=True)
            
            st.info(f"**Suggestion:** {suggestion}")
            
            # Display sentiment breakdown
            st.subheader("Sentiment Breakdown")
            
            # Count by category
            sentiment_counts = df['sentiment_category'].value_counts().reset_index()
            sentiment_counts.columns = ['Category', 'Count']
            
            # Create a horizontal bar chart
            st.bar_chart(sentiment_counts.set_index('Category'))
            
            # Display sentiment by feedback type
            st.subheader("Sentiment by Feedback Type")
            
            # Group by feedback type and calculate mean sentiment
            feedback_type_sentiment = df.groupby('type')['sentiment_score'].mean().reset_index()
            feedback_type_sentiment.columns = ['Feedback Type', 'Average Sentiment']
            
            # Add sentiment category column
            feedback_type_sentiment['Sentiment Category'] = feedback_type_sentiment['Average Sentiment'].apply(get_sentiment_category)
            
            # Display as a table
            st.table(feedback_type_sentiment)
            
            # Display top positive and negative feedback
            if len(df) > 1:
                st.subheader("Top Positive and Negative Feedback")
                
                col1, col2 = st.columns(2)
                
                with col1:
                    st.write("### Most Positive")
                    top_positive = df.sort_values('sentiment_score', ascending=False).iloc[0]
                    st.markdown(f"""
                    <div style="border-left: 5px solid green; padding-left: 10px;">
                        <p><strong>{top_positive['team_member']}</strong> - Score: {top_positive['sentiment_score']:.2f}</p>
                        <p>{top_positive['text']}</p>
                    </div>
                    """, unsafe_allow_html=True)
                
                with col2:
                    st.write("### Most Negative")
                    top_negative = df.sort_values('sentiment_score', ascending=True).iloc[0]
                    st.markdown(f"""
                    <div style="border-left: 5px solid red; padding-left: 10px;">
                        <p><strong>{top_negative['team_member']}</strong> - Score: {top_negative['sentiment_score']:.2f}</p>
                        <p>{top_negative['text']}</p>
                    </div>
                    """, unsafe_allow_html=True)
            
        else:
            st.info("No feedback added yet for this sprint. Add some feedback to see sentiment analysis.")
else:
    # Display welcome message when no board is selected
    st.write("## Welcome to the Agile Retrospective Sentiment Analyzer!")
    st.write("""
    This tool helps your team capture and analyze feedback from sprint retrospectives.
    
    ### Features:
    - Create boards for each sprint
    - Submit categorized feedback
    - Automatic sentiment analysis
    - Visualization of team sentiment
    - Export data for further analysis
    
    ### Get Started:
    Create a new retrospective board using the sidebar on the left.
    """)
    
    # Sample image for illustration
    st.image("https://via.placeholder.com/800x400?text=Agile+Retrospective+Visualization", 
             caption="Example visualization of retrospective feedback analysis")
