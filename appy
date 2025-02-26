import openai
import streamlit as st
import re
from datetime import datetime

def extract_date(clinic_note):
    """Extracts the first date found in MM/DD/YYYY format or converts textual date to this format."""
    date_patterns = [
        r"\b(\d{1,2}/\d{1,2}/\d{4})\b",  # Matches MM/DD/YYYY
        r"\b(January|February|March|April|May|June|July|August|September|October|November|December)\s(\d{1,2}),\s(\d{4})\b"
    ]
    for pattern in date_patterns:
        match = re.search(pattern, clinic_note)
        if match:
            if len(match.groups()) == 1:
                return match.group(1)  # MM/DD/YYYY format
            else:
                # Convert 'Month DD, YYYY' to MM/DD/YYYY
                month = datetime.strptime(match.group(1), "%B").month
                return f"{month:02d}/{int(match.group(2)):02d}/{match.group(3)}"
    return "[Date Not Found]"

def extract_ap_section(clinic_note):
    """Extracts the Assessment and Plan (A/P) section from the clinic note."""
    ap_match = re.search(r"Assessment and Plan[:\n](.*?)(?:\n\n|$)", clinic_note, re.DOTALL | re.IGNORECASE)
    return ap_match.group(1).strip() if ap_match else "[A/P Not Found]"

def summarize_hpi(clinic_note):
    """Function to summarize HPI from a clinic note using OpenAI API."""
    visit_date = extract_date(clinic_note)
    ap_section = extract_ap_section(clinic_note)
    response = openai.ChatCompletion.create(
        model="gpt-4-turbo",
        messages=[
            {"role": "system", "content": "You are a medical assistant specializing in urology documentation. Summarize the HPI from prior clinic notes into a structured, concise format with 4-7 bullet points, avoiding bold text, for easy pasting into the EMR. Use relevant details from the Assessment and Plan (A/P) section to update the HPI for today's visit. Start the summary with the date in MM/DD/YYYY format when the patient initially saw the provider. End the summary with a statement that the patient is being seen today for follow-up."},
            {"role": "user", "content": f"Summarize the HPI using the following clinic note and incorporate relevant updates from the A/P: {clinic_note}\nA/P Section: {ap_section}"},
        ]
    )
    summary_lines = response["choices"][0]["message"]["content"].split("\n")
    formatted_summary = f"Initial Visit: {visit_date}\n" + "\n".join([f"- {line}" for line in summary_lines if line.strip()])
    return formatted_summary + "\n- Patient is being seen today for follow-up."

# Streamlit UI
st.title("Urology AI Chatbot - HPI Summarizer")
st.write("Paste a prior clinic note, and I will generate a concise HPI summary incorporating updates from the A/P section.")

# User input
clinic_note = st.text_area("Enter prior clinic note:")

if clinic_note:
    summary = summarize_hpi(clinic_note)
    st.write("HPI Summary:")
    st.text(summary)
