import streamlit as st
import pandas as pd

st.title("📊 Student Marks Dashboard")

if 'students' not in st.session_state:
    st.session_state.students = []

# Sidebar inputs
st.sidebar.header("➕ Add Student")
name = st.sidebar.text_input("Student Name")
marks = st.sidebar.number_input("Marks (0-100)", min_value=0, max_value=100)

if st.sidebar.button("Add"):
    if name:
        st.session_state.students.append({'Name': name, 'Marks': marks})
        st.sidebar.success(f"✅ Added {name}")
    else:
        st.sidebar.warning("⚠️ Enter a valid name.")

# --- New: Search/Filter ---
search = st.text_input("🔍 Search Student by Name")
sort_order = st.radio("Sort by Marks", ["Descending", "Ascending"], horizontal=True)

# Main section
st.header("🧑‍🎓 Student List")
if st.session_state.students:
    df = pd.DataFrame(st.session_state.students)

    # --- New: Add Grade Column ---
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

    # --- New: Filter by search ---
    if search:
        df = df[df['Name'].str.contains(search, case=False, na=False)]

    # --- New: Sort by marks ---
    df = df.sort_values('Marks', ascending=(sort_order == "Ascending")).reset_index(drop=True)

    # Delete & Edit button for each row
    for i in range(len(df)):
        col1, col2, col3 = st.columns([6, 1, 1])
        with col1:
            st.write(f"{df.iloc[i]['Name']} — {df.iloc[i]['Marks']} marks — Grade: {df.iloc[i]['Grade']}")
        with col2:
            if st.button("✏️", key=f"edit_{i}"):
                new_marks = st.number_input(f"Edit marks for {df.iloc[i]['Name']}", min_value=0, max_value=100, value=int(df.iloc[i]['Marks']), key=f"edit_marks_{i}")
                if st.button("Save", key=f"save_{i}"):
                    # Find the index in the original session_state list
                    orig_idx = next(idx for idx, s in enumerate(st.session_state.students) if s['Name'] == df.iloc[i]['Name'] and s['Marks'] == df.iloc[i]['Marks'])
                    st.session_state.students[orig_idx]['Marks'] = new_marks
                    st.experimental_rerun()
        with col3:
            if st.button("❌", key=f"delete_{i}"):
                # Find the index in the original session_state list
                orig_idx = next(idx for idx, s in enumerate(st.session_state.students) if s['Name'] == df.iloc[i]['Name'] and s['Marks'] == df.iloc[i]['Marks'])
                st.session_state.students.pop(orig_idx)
                st.experimental_rerun()

    df = pd.DataFrame(st.session_state.students)
    df['Grade'] = df['Marks'].apply(get_grade)

    # Display DataFrame and Stats
    st.subheader("📋 Table")
    st.dataframe(df)

    st.subheader("📈 Statistics")
    st.write(f"Average: {df['Marks'].mean():.2f}")
    st.write(f"Max: {df['Marks'].max()}")
    st.write(f"Min: {df['Marks'].min()}")

    # Show bar chart
    st.subheader("📊 Marks Bar Chart")
    st.bar_chart(df.set_index('Name')['Marks'])

    # --- New: Pie Chart for Grades ---
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
        st.experimental_rerun()
else:
    st.info("No data yet. Add some students!")

