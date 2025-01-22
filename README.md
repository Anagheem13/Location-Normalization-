import pandas as pd
from googletrans import Translator
import re

# Initialize the translator
translator = Translator()

# Read the existing Excel file with city names, links, and city codes
input_file = "city_links_with_codes.xlsx"
df = pd.read_excel(input_file)


# Function to handle abbreviations
def handle_abbreviation(city_name):
    # Detect abbreviations in the format ת״א (letters with " between them)
    if re.match(r"^[א-ת]\u05F4[א-ת]$", city_name):  # Match Hebrew letters with a Hebrew-style double quote (״)
        # Create a fallback dictionary for known abbreviations
        abbreviation_mapping = {
            "ת״א": "Tel Aviv",
            "ר״ג": "Ramat Gan"
        }
        # Return the mapped full name if available
        if city_name in abbreviation_mapping:
            return abbreviation_mapping[city_name]

         #If not in the dictionary, expand dynamically
        first_letter = city_name[0]  # First letter
        second_letter = city_name[-1]  # Second letter
        expanded_name = f"{first_letter} {second_letter}"

        # Attempt to translate the expanded name
        try:
            translation = translator.translate(expanded_name, src='auto', dest='en')
            return translation.text
        except Exception as e:
            print(f"Error translating abbreviation {city_name}: {e}")
            return None
    return city_name


# Function to preprocess city names
def preprocess_city_name(city_name):
    # Handle abbreviations dynamically
    city_name = handle_abbreviation(city_name)

    # Remove numbers and special characters (retain only letters and spaces)
    clean_name = re.sub(r"[^\w\s]", "", city_name)  # Remove non-alphanumeric characters
    clean_name = re.sub(r"\d", "", clean_name)  # Remove digits
    return clean_name.strip()


# Function to translate city names to English
def translate_to_english(city_name):
    try:
        # Preprocess the city name
        preprocessed_name = preprocess_city_name(city_name)

        # Translate the preprocessed name to English
        translation = translator.translate(preprocessed_name, src='auto', dest='en')
        return translation.text
    except Exception as e:
        print(f"Error translating {city_name}: {e}")
        return None


# Function to translate English to Hebrew
def translate_to_hebrew(english_name):
    try:
        # Translate the English name back to Hebrew
        translation = translator.translate(english_name, src='en', dest='he')
        return translation.text
    except Exception as e:
        print(f"Error translating {english_name} to Hebrew: {e}")
        return None


# Function to get the city code from the Excel DataFrame based on the city name
def get_city_code_from_df(city_name):
    # Search for the city name in the DataFrame and return the city code
    matching_row = df[df['City Name'].str.contains(city_name, case=False, na=False)]
    if not matching_row.empty:
        return matching_row.iloc[0]['City Code']  # Return the first matching city code
    else:
        return "Not Found"  # If no match is found


# Function to process the list of city names and include city codes
def process_city_names(city_list):
    results = []
    for city in city_list:
        # Translate the city name to English
        translated_city = translate_to_english(city)

        if translated_city:
            # Now translate the English name to Hebrew
            translated_to_hebrew = translate_to_hebrew(translated_city)

            # Get the city code from the DataFrame
            city_code = get_city_code_from_df(translated_city)

            results.append({
                "Original City Name": city,
                "Translated City Name (English)": translated_city,
                "Translated City Name (Hebrew)": translated_to_hebrew,
                "City Code": city_code,
            })
    return results


# Example input list of city names in any language
city_names_input = ["ת״א", "ר״ג", "תל-אביב", "00تل ابيب","סכנין"]

# Process the city names
processed_results = process_city_names(city_names_input)

# Display the results
for result in processed_results:
    print(f"Original City: {result['Original City Name']}")
    print(f"Translated City (English): {result['Translated City Name (English)']}")
    print(f"Translated City (Hebrew): {result['Translated City Name (Hebrew)']}")
    print(f"City Code: {result['City Code']}")
    print("-" * 50)
