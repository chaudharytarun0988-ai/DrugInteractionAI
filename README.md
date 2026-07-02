import streamlit as st
import json
from datetime import datetime, timedelta

# ============================================================
# DRUG DATABASE - Add your own drugs here!
# ============================================================
drug_db = {
    "Aspirin": {
        "name": {"en": "Aspirin", "hi": "एस्पिरिन"},
        "min_age": 12,
        "kidney_risk": False,
        "liver_risk": False,
        "pregnancy_risk": "C",
        "breastfeeding_risk": "Caution",
        "allergens": [],
        "interactions": {
            "Warfarin": {"severity": "High", "desc": "Increased risk of bleeding"}
        }
    },
    "Paracetamol": {
        "name": {"en": "Paracetamol", "hi": "पैरासिटामोल"},
        "min_age": 1,
        "kidney_risk": False,
        "liver_risk": True,
        "pregnancy_risk": "B",
        "breastfeeding_risk": "Safe",
        "allergens": [],
        "interactions": {
            "Warfarin": {"severity": "Medium", "desc": "May increase INR"}
        }
    },
    "Warfarin": {
        "name": {"en": "Warfarin", "hi": "वारफारिन"},
        "min_age": 18,
        "kidney_risk": True,
        "liver_risk": True,
        "pregnancy_risk": "X",
        "breastfeeding_risk": "Contraindicated",
        "allergens": [],
        "interactions": {
            "Aspirin": {"severity": "High", "desc": "Severe bleeding risk"},
            "Paracetamol": {"severity": "Medium", "desc": "May affect INR"}
        }
    },
    "Ibuprofen": {
        "name": {"en": "Ibuprofen", "hi": "इबुप्रोफेन"},
        "min_age": 6,
        "kidney_risk": True,
        "liver_risk": False,
        "pregnancy_risk": "D",
        "breastfeeding_risk": "Caution",
        "allergens": [],
        "interactions": {
            "Warfarin": {"severity": "High", "desc": "Increased bleeding risk"},
            "Aspirin": {"severity": "Medium", "desc": "Reduced antiplatelet effect"}
        }
    },
    "Amoxicillin": {
        "name": {"en": "Amoxicillin", "hi": "एमोक्सिसिलिन"},
        "min_age": 0,
        "kidney_risk": True,
        "liver_risk": False,
        "pregnancy_risk": "B",
        "breastfeeding_risk": "Safe",
        "allergens": ["Penicillin"],
        "interactions": {}
    },
    "Metformin": {
        "name": {"en": "Metformin", "hi": "मेटफॉर्मिन"},
        "min_age": 10,
        "kidney_risk": True,
        "liver_risk": False,
        "pregnancy_risk": "B",
        "breastfeeding_risk": "Safe",
        "allergens": [],
        "interactions": {}
    }
}

# ============================================================
# MAIN APP
# ============================================================
st.set_page_config(page_title="Drug Interaction Checker", page_icon="💊", layout="centered")

# Language selection
lang = st.sidebar.radio("Language / भाषा", ["English", "हिंदी"])
st.sidebar.markdown("---")
st.sidebar.markdown("👨‍⚕️ **Pharma Final Year Project**")
st.sidebar.markdown("📚 *Add more drugs in the database*")

# Language helper
def t(en, hi):
    return hi if lang == "हिंदी" else en

st.title("💊 " + t("Drug Interaction Checker", "दवा इंटरैक्शन चेकर"))

# ============================================================
# PATIENT DETAILS
# ============================================================
st.subheader(t("👤 Patient Details", "👤 मरीज की जानकारी"))

col1, col2 = st.columns(2)
with col1:
    age = st.number_input(t("Age (years)", "उम्र (साल)"), min_value=0, max_value=120, value=30, step=1)
with col2:
    weight = st.number_input(t("Weight (kg) - Optional", "वजन (किग्रा) - वैकल्पिक"), min_value=0, max_value=300, value=70, step=1)

st.markdown("---")

# ============================================================
# MEDICAL CONDITIONS (Checkboxes)
# ============================================================
st.subheader(t("🩺 Medical Conditions", "🩺 चिकित्सीय स्थितियाँ"))

col1, col2, col3, col4 = st.columns(4)
with col1:
    kidney = st.checkbox(t("Kidney Disease", "किडनी रोग"))
with col2:
    liver = st.checkbox(t("Liver Disease", "लिवर रोग"))
with col3:
    pregnant = st.checkbox(t("Pregnant", "गर्भवती"))
with col4:
    breastfeeding = st.checkbox(t("Breastfeeding", "स्तनपान"))

allergy_pen = st.checkbox(t("Penicillin Allergy", "पेनिसिलिन एलर्जी"))

st.markdown("---")

# ============================================================
# DRUG SELECTION
# ============================================================
st.subheader(t("💊 Add Medications", "💊 दवाएँ जोड़ें"))

# Initialize session state for selected drugs
if "selected_drugs" not in st.session_state:
    st.session_state.selected_drugs = []

col1, col2 = st.columns([3, 1])
with col1:
    drug_options = list(drug_db.keys())
    new_drug = st.selectbox(t("Select a drug", "दवा चुनें"), [""] + drug_options)
with col2:
    if st.button(t("➕ Add", "➕ जोड़ें"), use_container_width=True):
        if new_drug and new_drug not in st.session_state.selected_drugs:
            st.session_state.selected_drugs.append(new_drug)
            st.rerun()
        elif new_drug in st.session_state.selected_drugs:
            st.warning(t("Drug already added!", "दवा पहले से जोड़ी गई है!"))

# Display selected drugs as tags
if st.session_state.selected_drugs:
    st.write(t("**Added Drugs:**", "**जोड़ी गई दवाएँ:**"))
    cols = st.columns(len(st.session_state.selected_drugs))
    for idx, drug in enumerate(st.session_state.selected_drugs):
        with cols[idx]:
            if st.button(f"❌ {drug}", key=f"remove_{drug}"):
                st.session_state.selected_drugs.remove(drug)
                st.rerun()
else:
    st.info(t("No drugs added yet. Select a drug above.", "अभी तक कोई दवा नहीं जोड़ी गई। ऊपर से दवा चुनें।"))

st.markdown("---")

# ============================================================
# DOSE REMINDER
# ============================================================
st.subheader(t("⏰ Dose Reminder", "⏰ डोज़ रिमाइंडर"))

col1, col2 = st.columns([2, 1])
with col1:
    dose_interval = st.number_input(t("Frequency (hours)", "अंतराल (घंटे)"), min_value=1, max_value=72, value=8, step=1)
with col2:
    if st.button(t("Set Reminder", "रिमाइंडर सेट करें"), use_container_width=True):
        next_dose = datetime.now() + timedelta(hours=dose_interval)
        st.success(t(f"✅ Next dose at: {next_dose.strftime('%I:%M %p')} (every {dose_interval} hours)",
                     f"✅ अगली खुराक: {next_dose.strftime('%I:%M %p')} (हर {dose_interval} घंटे में)"))

st.markdown("---")

# ============================================================
# CHECK INTERACTIONS BUTTON
# ============================================================
if st.button(t("🔍 Check Interactions", "🔍 इंटरैक्शन चेक करें"), type="primary", use_container_width=True):
    st.markdown("---")
    st.subheader(t("📋 Results", "📋 परिणाम"))

    if not st.session_state.selected_drugs:
        st.warning(t("Please add at least one drug first!", "कृपया कम से कम एक दवा पहले जोड़ें!"))
    else:
        warnings = []
        has_warning = False

        # Check each drug
        for drug in st.session_state.selected_drugs:
            info = drug_db.get(drug)
            if not info:
                warnings.append(f"⚠️ {t('Data for', 'डेटा नहीं मिली')} '{drug}' {t('not found.', 'के लिए')}")
                continue

            # Age warning
            if age and info.get("min_age") and age < info["min_age"]:
                warnings.append(f"🔞 {drug}: {t('Not recommended below', 'से कम उम्र में सलाह नहीं')} {info['min_age']} {t('years.', 'साल।')}")
                has_warning = True

            # Kidney warning
            if kidney and info.get("kidney_risk"):
                warnings.append(f"🫘 {drug}: {t('Use with caution in Kidney Disease.', 'किडनी रोग में सावधानी बरतें।')}")
                has_warning = True

            # Liver warning
            if liver and info.get("liver_risk"):
                warnings.append(f"🫁 {drug}: {t('Use with caution in Liver Disease.', 'लिवर रोग में सावधानी बरतें।')}")
                has_warning = True

            # Pregnancy warning
            if pregnant:
                risk = info.get("pregnancy_risk")
                if risk == "X":
                    warnings.append(f"🚫 {drug}: {t('CONTRAINDICATED in pregnancy (Category X).', 'गर्भावस्था में वर्जित (श्रेणी X)।')}")
                    has_warning = True
                elif risk == "D":
                    warnings.append(f"⚠️ {drug}: {t('Avoid in pregnancy (Category D).', 'गर्भावस्था में टालें (श्रेणी D)।')}")
                    has_warning = True

            # Breastfeeding warning
            if breastfeeding:
                risk = info.get("breastfeeding_risk")
                if risk == "Contraindicated":
                    warnings.append(f"🚫 {drug}: {t('Contraindicated in breastfeeding.', 'स्तनपान में वर्जित।')}")
                    has_warning = True
                elif risk == "Caution":
                    warnings.append(f"⚠️ {drug}: {t('Use with caution in breastfeeding.', 'स्तनपान में सावधानी बरतें।')}")
                    has_warning = True

            # Penicillin allergy
            if allergy_pen and "Penicillin" in info.get("allergens", []):
                warnings.append(f"🤧 {drug}: {t('Penicillin allergy alert!', 'पेनिसिलिन एलर्जी की चेतावनी!')}")
                has_warning = True

        # Check drug-drug interactions
        for i in range(len(st.session_state.selected_drugs)):
            for j in range(i + 1, len(st.session_state.selected_drugs)):
                drug1 = st.session_state.selected_drugs[i]
                drug2 = st.session_state.selected_drugs[j]
                info1 = drug_db.get(drug1, {})

                if drug2 in info1.get("interactions", {}):
                    inter = info1["interactions"][drug2]
                    warnings.append(f"🔴 {drug1} + {drug2}: {inter['desc']} ({t('Severity', 'गंभीरता')}: {inter['severity']})")
                    has_warning = True

                # Check reverse
                info2 = drug_db.get(drug2, {})
                if drug1 in info2.get("interactions", {}):
                    inter = info2["interactions"][drug1]
                    warnings.append(f"🔴 {drug2} + {drug1}: {inter['desc']} ({t('Severity', 'गंभीरता')}: {inter['severity']})")
                    has_warning = True

        # Display results
        if not has_warning:
            st.success(t("✅ No major interactions or warnings found for these drugs based on the given patient data.",
                         "✅ इन दवाओं और मरीज की जानकारी के आधार पर कोई बड़ी इंटरैक्शन या चेतावनी नहीं मिली।"))
        else:
            for warning in warnings:
                st.warning(warning)

        st.caption(t("⚠️ *This is a demo project. Please consult a doctor before taking any medicine.*",
                     "⚠️ *यह एक डेमो प्रोजेक्ट है। कोई भी दवा लेने से पहले डॉक्टर से सलाह लें।*"))# DrugInteractionAI
AI-powered Drug Interaction Checker using Machine Learning and LLMs.
