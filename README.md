# Gradesheet-generator

import streamlit as st
from PIL import Image
import io

# Page Configuration
st.set_page_config(
    page_title="Student Grade Sheet",
    page_icon="📝",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Constants
MAX_PHOTO_SIZE = (150, 150)  # Maximum width and height in pixels
COMPULSORY_SUBJECTS = ["ENGLISH", "NEPALI", "SCIENCE", "MATH", "SOCIAL"]
OPTIONAL_SUBJECTS = ["Optional Mathematics", "Computer", "Health", "General Knowledge"]

def calculate_grade(mark):
    """Calculate grade based on marks."""
    if mark > 90: return "A+", "green"
    if mark > 80: return "A", "blue"
    if mark > 70: return "B+", "blue"
    if mark > 60: return "B", "blue"
    if mark > 50: return "C+", "blue"
    if mark > 40: return "C", "blue"
    if mark > 35: return "D", "blue"
    return "F", "red"

def resize_image(image, max_size):
    """Resize image while maintaining aspect ratio."""
    img = Image.open(image)
    img.thumbnail(max_size)
    return img

def initialize_session():
    """Initialize session state."""
    if 'form_data' not in st.session_state:
        st.session_state.form_data = None

def apply_styles():
    """Apply custom CSS styles."""
    st.markdown("""
    <style>
        .student-photo {
            max-width: 120px;
            border-radius: 5px;
            border: 1px solid #ddd;
            margin-bottom: 10px;
        }
        .compact-input {
            margin-bottom: 0.5rem;
        }
        .subject-row {
            margin-bottom: 0.5rem;
        }
    </style>
    """, unsafe_allow_html=True)

def student_info_section():
    """Collect student information."""
    st.subheader("Student Details")
    cols = st.columns([4, 1])
    
    with cols[0]:
        name = st.text_input("Full Name", key="name", placeholder="Enter student name")
        class_info = st.text_input("Class", key="class", placeholder="Enter class")
        
        sub_cols = st.columns(2)
        with sub_cols[0]:
            section = st.text_input("Section", key="section", placeholder="Enter section")
        with sub_cols[1]:
            roll_no = st.text_input("Roll No.", key="roll_no", placeholder="Enter roll number")
    
    with cols[1]:
        photo = st.file_uploader("Photo (120x120px)", type=['jpg', 'jpeg', 'png'], 
                                key="photo", help="Upload a square photo for best results")
        if photo:
            try:
                img = resize_image(photo, MAX_PHOTO_SIZE)
                st.image(img, use_column_width=True, caption="Preview")
            except Exception as e:
                st.error("Error processing image")
    
    return {
        'name': name,
        'class': class_info,
        'section': section,
        'roll_no': roll_no,
        'photo': photo
    }

def school_info_section():
    """Collect school information."""
    st.subheader("School Information")
    school = st.text_input("School Name", key="school", placeholder="Enter school name")
    board = st.text_input("Board Name", key="board", placeholder="Enter board name")
    address = st.text_input("Address", key="address", placeholder="Enter school address")
    
    return {
        'school': school,
        'board': board,
        'address': address
    }

def marks_section():
    """Collect subject marks."""
    st.subheader("Subject Marks")
    grades = []
    total_marks = 0
    valid_subjects = 0
    
    # Compulsory Subjects
    for subject in COMPULSORY_SUBJECTS:
        mark = st.number_input(
            f"{subject} Marks",
            min_value=0.0,
            max_value=100.0,
            step=0.5,
            key=f"marks_{subject}",
            value=None,
            placeholder="Enter marks"
        )
        if mark is not None:
            grade, color = calculate_grade(mark)
            grades.append({
                'subject': subject,
                'mark': mark,
                'grade': grade,
                'color': color
            })
            valid_subjects += 1
            total_marks += mark
    
    # Optional Subjects
    for i, subject in enumerate(OPTIONAL_SUBJECTS[:2]):
        cols = st.columns([3, 1])
        with cols[0]:
            selected = st.checkbox(f"{subject}", key=f"opt_subj_{i}")
        with cols[1]:
            if selected:
                mark = st.number_input(
                    "Marks",
                    min_value=0.0,
                    max_value=100.0,
                    step=0.5,
                    key=f"opt_marks_{i}",
                    value=None,
                    placeholder="Enter marks"
                )
                if mark is not None:
                    grade, color = calculate_grade(mark)
                    grades.append({
                        'subject': subject,
                        'mark': mark,
                        'grade': grade,
                        'color': color
                    })
                    valid_subjects += 1
                    total_marks += mark
    
    return grades, valid_subjects, total_marks

def validate_inputs(student, school, subjects):
    """Validate form inputs."""
    errors = []
    if not student['name']: errors.append("Student name is required")
    if not student['class']: errors.append("Class is required")
    if not student['roll_no']: errors.append("Roll number is required")
    if not school['school']: errors.append("School name is required")
    if subjects == 0: errors.append("At least one subject mark is required")
    return errors

def display_results(data):
    """Display the final results."""
    st.success(f"Grade Report for {data['student']['name']}")
    
    # Create two columns for layout
    main_col, photo_col = st.columns([4, 1])
    
    with main_col:
        # Student and School Info
        with st.expander("Personal Details", expanded=True):
            cols = st.columns(2)
            with cols[0]:
                st.write(f"**Name:** {data['student']['name']}")
                st.write(f"**Class:** {data['student']['class']}")
            with cols[1]:
                st.write(f"**Section:** {data['student']['section']}")
                st.write(f"**Roll No.:** {data['student']['roll_no']}")
            
            st.write(f"**School:** {data['school']['school']}")
            st.write(f"**Board:** {data['school']['board']}")
        
        # Subject Grades
        st.subheader("Subject-wise Grades")
        for grade in data['grades']:
            cols = st.columns([3, 1, 1])
            with cols[0]: st.write(grade['subject'])
            with cols[1]: st.write(f"{grade['mark']:.1f}")
            with cols[2]: 
                st.markdown(f"<span style='color:{grade['color']};font-weight:bold'>{grade['grade']}</span>", 
                           unsafe_allow_html=True)
        
        # Final Results
        st.subheader("Overall Performance")
        percentage = (data['total_marks'] / (data['valid_subjects'] * 100)) * 100
        final_grade, _ = calculate_grade(percentage)
        
        cols = st.columns(3)
        with cols[0]:
            st.metric("Total Marks", f"{data['total_marks']:.1f}/{data['valid_subjects'] * 100}")
        with cols[1]:
            st.metric("Percentage", f"{percentage:.1f}%")
        with cols[2]:
            st.metric("Final Grade", final_grade)
    
    with photo_col:
        if data['student']['photo']:
            try:
                img = resize_image(data['student']['photo'], MAX_PHOTO_SIZE)
                st.image(img, caption="Student Photo", width=120)
            except:
                st.warning("Couldn't display photo")

def main():
    """Main application function."""
    st.title("Student Grade Sheet")
    initialize_session()
    apply_styles()
    
    with st.form("grade_form"):
        student = student_info_section()
        school = school_info_section()
        grades, valid_subjects, total_marks = marks_section()
        
        if st.form_submit_button("Generate Report"):
            errors = validate_inputs(student, school, valid_subjects)
            if errors:
                for error in errors:
                    st.error(error)
            else:
                st.session_state.form_data = {
                    'student': student,
                    'school': school,
                    'grades': grades,
                    'valid_subjects': valid_subjects,
                    'total_marks': total_marks
                }
    
    if st.session_state.form_data:
        display_results(st.session_state.form_data)

if __name__ == "__main__":
    main()
