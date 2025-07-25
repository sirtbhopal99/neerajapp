

import streamlit as st
import pandas as pd
import time

st.set_page_config(page_title="Student Marks Dashboard", page_icon=":bar_chart:", layout="centered")

st.title("📊 Student Marks Dashboard")

if 'students' not in st.session_state:
    st.session_state.students = []
if 'confetti' not in st.session_state:
    st.session_state.confetti = False

# Sidebar inputs
st.sidebar.header("➕ Add Student")
name = st.sidebar.text_input("Student Name")
marks = st.sidebar.number_input("Marks (0-100)", min_value=0, max_value=100)

if st.sidebar.button("Add"):
    if name:
        st.session_state.students.append({'Name': name, 'Marks': marks})
        st.session_state.confetti = True
        st.sidebar.success(f"✅ Added {name}")
        st.toast(f"🎉 Welcome, {name}!", icon="🎈")
    else:
        st.sidebar.warning("⚠️ Enter a valid name.")

# Confetti animation when a student is added
if st.session_state.confetti:
    st.balloons()
    st.session_state.confetti = False

# --- Search/Filter ---
search = st.text_input("🔍 Search Student by Name")
sort_order = st.radio("Sort by Marks", ["Descending", "Ascending"], horizontal=True)

# Main section
st.header("🧑‍🎓 Student List")
if st.session_state.students:
    df = pd.DataFrame(st.session_state.students)

    # Add Grade and Pass/Fail
    def get_grade(m):
        if m >= 90:
            return "A"
        elif m >= 75:
            return "B"
        elif m >= 60:
            return "C"
        elif m >= 40:
            return "D"
        else:
            return "F"
    df['Grade'] = df['Marks'].apply(get_grade)
    df['Status'] = df['Marks'].apply(lambda x: "✅ Pass" if x >= 40 else "❌ Fail")

    # Filter by search
    if search:
        df = df[df['Name'].str.contains(search, case=False, na=False)]

    # Sort by marks
    df = df.sort_values('Marks', ascending=(sort_order == "Ascending")).reset_index(drop=True)

    # Highlight topper(s)
    max_marks = df['Marks'].max()
    toppers = df[df['Marks'] == max_marks]['Name'].tolist()

    # Student count
    st.markdown(f"**👥 Total Students:** {len(df)}")
    st.markdown(f"🏆 **Topper(s):** <span style='color:gold'>{', '.join(toppers)}</span> with <b>{max_marks}</b> marks", unsafe_allow_html=True)

    # Progress bar for average marks
    avg = df['Marks'].mean()
    st.markdown(f"**Average Marks:** {avg:.2f}")
    st.progress(int(avg))

    # Animated table with colored status
    for i in range(len(df)):
        col1, col2, col3, col4, col5 = st.columns([4, 2, 1, 1, 1])
        with col1:
            st.write(f"**{df.iloc[i]['Name']}**")
        with col2:
            st.write(f"{df.iloc[i]['Marks']} marks")
        with col3:
            st.write(f"Grade: {df.iloc[i]['Grade']}")
        with col4:
            color = "green" if df.iloc[i]['Status'] == "✅ Pass" else "red"
            st.markdown(f"<span style='color:{color}'>{df.iloc[i]['Status']}</span>", unsafe_allow_html=True)
        with col5:
            if st.button("✏️", key=f"edit_{i}"):
                new_marks = st.number_input(f"Edit marks for {df.iloc[i]['Name']}", min_value=0, max_value=100, value=int(df.iloc[i]['Marks']), key=f"edit_marks_{i}")
                if st.button("Save", key=f"save_{i}"):
                    orig_idx = next(idx for idx, s in enumerate(st.session_state.students) if s['Name'] == df.iloc[i]['Name'] and s['Marks'] == df.iloc[i]['Marks'])
                    st.session_state.students[orig_idx]['Marks'] = new_marks
                    st.experimental_rerun()
            if st.button("❌", key=f"delete_{i}"):
                orig_idx = next(idx for idx, s in enumerate(st.session_state.students) if s['Name'] == df.iloc[i]['Name'] and s['Marks'] == df.iloc[i]['Marks'])
                st.session_state.students.pop(orig_idx)
                st.toast(f"🗑️ Deleted {df.iloc[i]['Name']}")
                st.experimental_rerun()

    # DataFrame and Stats
    st.subheader("📋 Table")
    st.dataframe(df, use_container_width=True)

    st.subheader("📈 Statistics")
    st.write(f"Average: {df['Marks'].mean():.2f}")
    st.write(f"Max: {df['Marks'].max()}")
    st.write(f"Min: {df['Marks'].min()}")

    # Bar chart
    st.subheader("📊 Marks Bar Chart")
    st.bar_chart(df.set_index('Name')['Marks'])

    # Pie Chart for Grades
    st.subheader("🟢 Grade Distribution")
    st.pyplot(df['Grade'].value_counts().plot.pie(autopct='%1.1f%%', ylabel='').get_figure())

    # Download CSV
    csv = df.to_csv(index=False).encode('utf-8')
    st.download_button(
        label="📥 Download CSV",
        data=csv,
        file_name='student_marks.csv',
        mime='text/csv'
    )

    # Clear all data
    if st.button("🗑️ Clear All"):
        st.session_state.students = []
        st.toast("All data cleared!", icon="🧹")
        st.experimental_rerun()
else:
    st.info("No data yet. Add some students!")

