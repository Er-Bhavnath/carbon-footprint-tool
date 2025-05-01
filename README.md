import matplotlib                                                                          #library to store the pdf without backend
matplotlib.use('Agg')                                                                      # Use a non-interactive backend
import matplotlib.pyplot as plt                                                            # library to create bar chart  and pie chart 
from fpdf import FPDF                                                                      # to create a pdf file
def get_valid_input(prompt, input_type=float, min_value=0, max_value=None):                
    """Get valid numeric input from the user."""
    while True:                                                                            #use while loop to re enter the input again and again until valid 
        try:
            value = input_type(input(f"{prompt}: "))
            if value < min_value:
                raise ValueError(f"Value must be greater than or equal to {min_value}.")
            if value == 0:
                print("Value cannot be zero. Please enter a valid number greater than zero.")
                continue
            if max_value is not None and value > max_value:
                raise ValueError(f"Value must be less than or equal to {max_value}.")
            return value
        except ValueError as e:
            print(f"Invalid input. {e} Please try again.")

def calculate_emissions():
    """Collect user input and calculate emissions."""
    print("\n=== Energy Consumption ===")
    electricity = get_valid_input("Enter monthly electricity usage (kWh)", float, 0)
    gas = get_valid_input("Enter monthly natural gas usage (therms)", float, 0)
    fuel = get_valid_input("Enter monthly fuel usage (liters)", float, 0)
    
    print("\n=== Transportation ===")
    travel_distance = get_valid_input("Enter monthly total travel distance (miles)", float, 0)
    fuel_efficiency = get_valid_input("Enter fuel efficiency (miles per Liter)", float, 0)
    
    print("\n=== Waste Management ===")
    waste_generated = get_valid_input("Enter monthly waste generated (kg)", float, 0)
    recycling_percent = get_valid_input("Enter recycling percentage (0-100)", float, 0, 100)
    
    # Convert all monthly data to yearly data (multiply by 12)
    electricity *= 12
    gas *= 12
    fuel *= 12
    travel_distance *= 12
    waste_generated *= 12

    # Emission calculations (in kg CO2 per year)
    energy_emissions = (electricity * 0.92 + gas * 5.3 + fuel * 2.31)          #formula to calculate co2 from electricity,gasand fule
    transportation_emissions = (travel_distance / fuel_efficiency) * 19.6
    waste_emissions = waste_generated * (1 - recycling_percent / 100) * 0.3

    # Convert emissions from kg to tons (1 ton = 1000 kg)
    energy_emissions /= 1000
    transportation_emissions /= 1000
    waste_emissions /= 1000

    return {
        "energy": energy_emissions,
        "transportation": transportation_emissions,
        "waste": waste_emissions
    }

def determine_emission_level(emission):
    """Determine emission level and return corresponding color and level text."""
    if emission > 3:
        return "High", (255, 0, 0)  # Red
    elif 1.5 <= emission <= 3:
        return "Medium", (255, 255, 0)  # Yellow
    else:
        return "Low", (0, 128, 0)  # Green

def provide_suggestions(emissions):
    """Provide suggestions based on emission categories (High, Medium, Low)."""
    suggestions = {}                                                                     #Add a dictionary

    # Suggestions for Energy Consumption
    energy_suggestions = []
    if emissions["energy"] > 4:
        energy_suggestions.append("#Your emissions are very high.Consider the following actions:-\n -Using energy-efficient appliances and LED lighting.")
        energy_suggestions.append("Switching to renewable energy sources like solar or wind.")
        energy_suggestions.append("Reducing overall energy consumption with smart home devices.")
    elif 1.5 <= emissions["energy"] <= 4:
        energy_suggestions.append("#Your emissions are moderate. You can further reduce them by:\n -Improving home insulation to save on heating/cooling energy.")
        energy_suggestions.append("Using appliances during off-peak hours to reduce strain on energy grids.")
    else:
        energy_suggestions.append("#Great job! Your emissions are low.\n Here are additional tips to maintain or improve:\n -Maintain efficiency by monitoring energy usage regularly.")
        energy_suggestions.append("Explore net-zero energy solutions like solar panels.")
    suggestions["energy"] = energy_suggestions

    # Suggestions for Transportation
    transportation_suggestions = []
    if emissions["transportation"] > 4:
        transportation_suggestions.append("#Your emissions are very high.Consider the following actions:-\n -Carpooling, using public transport, or biking more frequently.")
        transportation_suggestions.append("Switching to hybrid or electric vehicles to reduce fuel use.")
        transportation_suggestions.append("Reducing long-distance travel whenever possible.")
    elif 1.5 <= emissions["transportation"] <= 4:
        transportation_suggestions.append("#Your emissions are moderate. You can further reduce them by:\n-Optimizing routes to save fuel.")
        transportation_suggestions.append("Combining errands to reduce overall mileage.")
    else:
        transportation_suggestions.append("#Great job! Your emissions are low. Here are additional tips to maintain or improve:\n-Keep up eco-friendly travel habits like walking or biking.")
        transportation_suggestions.append("Encourage others to adopt sustainable travel options.")
    suggestions["transportation"] = transportation_suggestions

    # Suggestions for Waste Management
    waste_suggestions = []
    if emissions["waste"] > 4:
        waste_suggestions.append("#Your emissions are very high.Consider the following actions:-\n -Increasing recycling efforts and composting organic waste.")
        waste_suggestions.append("Avoiding single-use plastics and opting for reusable products.")
        waste_suggestions.append("Reducing food waste by planning meals and proper storage.")
    elif 1.5 <= emissions["waste"] <= 4:
        waste_suggestions.append("#Your emissions are moderate. You can further reduce them by:\n -Expanding recycling habits (e.g., e-waste and paper).")
        waste_suggestions.append("Donating items instead of discarding them.")
    else:
        waste_suggestions.append("#Great job! Your emissions are low. Here are additional tips to maintain or improve:\n -Maintain minimal waste by reducing, reusing, and recycling.")
        waste_suggestions.append("Continue composting organic waste to keep emissions low.")
    suggestions["waste"] = waste_suggestions

    return suggestions

    

def visualize_data(emissions):
    """Visualize emissions data and save the charts."""
    categories = list(emissions.keys())
    values = list(emissions.values())
    
    # Pie Chart
    plt.figure(figsize=(10, 5))
    plt.subplot(1, 2, 1)
    plt.pie(values, labels=categories, autopct="%1.1f%%", startangle=140)
    plt.title("Overall Emissions Distribution (tons)")
    plt.savefig("pie_chart.png")

    # Bar Chart
    plt.subplot(1, 2, 2)
    plt.bar(categories, values, color=['blue', 'green', 'orange'])
    plt.title("Emissions by Category (tons)")
    plt.ylabel("CO2 Emissions (tons)")
    plt.xlabel("Categories")
    plt.tight_layout()
    plt.savefig("bar_chart.png")
    plt.close()

def generate_pdf_report(emissions, suggestions):
    """Generate PDF report with emissions data and visualizations."""
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", 'B', 16)
    pdf.cell(0, 10, "CARBON FOOTPRINT REPORT", ln=True, align='C')
    pdf.ln(10)
    
    # Emissions Breakdown
    pdf.set_font("Arial", size=12)
    pdf.cell(0, 10, "=== Emissions Breakdown (Annual, in Tons) ===", ln=True)
    for category, value in emissions.items():
        pdf.cell(0, 10, f"{category.capitalize()} Emissions: {value:.2f} tons CO2", ln=True)
    
    pdf.ln(5)
    
    # Suggestions with color
    for category, lines in suggestions.items():
        level, color = determine_emission_level(emissions[category])
        pdf.set_text_color(*color)  # Set color dynamically
        pdf.cell(0, 10, f"{category.capitalize()} Emissions - {level}:", ln=True)
        pdf.set_text_color(0, 0, 0)  # Reset to black
        for line in lines:
            pdf.multi_cell(0, 10, f"  - {line}")
        pdf.ln(2)
    
    pdf.ln(10)
    
    # Add charts
    pdf.cell(0, 10, "  EMISSIONS VISUALIZATION:", ln=True)
    pdf.image("pie_chart.png", x=10, y=pdf.get_y(), w=90)
    pdf.image("bar_chart.png", x=110, y=pdf.get_y(), w=90)
    pdf.output("carbon_footprint_report.pdf")
    print("\nPDF Report Generated: 'carbon_footprint_report.pdf'")

def main():
    print("Welcome to the Carbon Emission Tracker!")
    emissions = calculate_emissions()
    print("\n Emissions Breakdown (Annual, in Tons)")
    for category, value in emissions.items():
        print(f"{category.capitalize()} Emissions: {value:.2f} tons CO2")
    
    suggestions = provide_suggestions(emissions)                                                # Now returns a dictionary
    visualize_data(emissions)
    generate_pdf_report(emissions, suggestions)


if __name__ == "__main__":
    main()
