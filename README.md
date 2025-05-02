import sqlite3
import os

class AnimalSpeciesDatabase:
    def __init__(self, db_name="animal_species.db"):
        """Initialize the database connection"""
        self.db_name = db_name
        self.connection = None
        self.cursor = None
        self.connect_to_db()
        self.create_tables()
    
    def connect_to_db(self):
        """Connect to the SQLite database"""
        try:
            self.connection = sqlite3.connect(self.db_name)
            self.cursor = self.connection.cursor()
            print(f"Connected to database: {self.db_name}")
        except sqlite3.Error as e:
            print(f"Error connecting to database: {e}")
    
    def create_tables(self):
        """Create the necessary tables if they don't exist"""
        try:
            self.cursor.execute('''
                CREATE TABLE IF NOT EXISTS species (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT NOT NULL,
                    classification TEXT NOT NULL,
                    habitat TEXT NOT NULL,
                    diet TEXT NOT NULL,
                    lifespan INTEGER,
                    description TEXT
                )
            ''')
            self.connection.commit()
            print("Tables created successfully")
        except sqlite3.Error as e:
            print(f"Error creating tables: {e}")
    
    def add_species(self, name, classification, habitat, diet, lifespan, description):
        """Add a new species to the database"""
        try:
            self.cursor.execute('''
                INSERT INTO species (name, classification, habitat, diet, lifespan, description)
                VALUES (?, ?, ?, ?, ?, ?)
            ''', (name, classification, habitat, diet, lifespan, description))
            self.connection.commit()
            print(f"Species '{name}' added successfully")
            return True
        except sqlite3.Error as e:
            print(f"Error adding species: {e}")
            return False
    
    def search_species(self, search_term=None, classification=None):
        """Search for species by name or classification"""
        try:
            query = "SELECT * FROM species WHERE 1=1"
            params = []
            
            if search_term:
                query += " AND (name LIKE ? OR description LIKE ?)"
                params.extend([f"%{search_term}%", f"%{search_term}%"])
            
            if classification and classification != "All":
                query += " AND classification = ?"
                params.append(classification)
            
            self.cursor.execute(query, params)
            results = self.cursor.fetchall()
            
            if not results:
                print("No species found matching your criteria")
                return []
            
            return results
        except sqlite3.Error as e:
            print(f"Error searching species: {e}")
            return []
    
    def get_all_species(self):
        """Get all species from the database"""
        try:
            self.cursor.execute("SELECT * FROM species")
            return self.cursor.fetchall()
        except sqlite3.Error as e:
            print(f"Error retrieving species: {e}")
            return []
    
    def get_species_by_id(self, species_id):
        """Get a specific species by ID"""
        try:
            self.cursor.execute("SELECT * FROM species WHERE id = ?", (species_id,))
            return self.cursor.fetchone()
        except sqlite3.Error as e:
            print(f"Error retrieving species: {e}")
            return None
    
    def update_species(self, species_id, name, classification, habitat, diet, lifespan, description):
        """Update an existing species"""
        try:
            self.cursor.execute('''
                UPDATE species
                SET name = ?, classification = ?, habitat = ?, diet = ?, lifespan = ?, description = ?
                WHERE id = ?
            ''', (name, classification, habitat, diet, lifespan, description, species_id))
            self.connection.commit()
            print(f"Species with ID {species_id} updated successfully")
            return True
        except sqlite3.Error as e:
            print(f"Error updating species: {e}")
            return False
    
    def delete_species(self, species_id):
        """Delete a species from the database"""
        try:
            self.cursor.execute("DELETE FROM species WHERE id = ?", (species_id,))
            self.connection.commit()
            print(f"Species with ID {species_id} deleted successfully")
            return True
        except sqlite3.Error as e:
            print(f"Error deleting species: {e}")
            return False
    
    def close_connection(self):
        """Close the database connection"""
        if self.connection:
            self.connection.close()
            print("Database connection closed")


def display_species(species_list):
    """Display species information in a formatted way"""
    if not species_list:
        print("No species to display")
        return
    
    print("\n" + "="*80)
    print(f"{'ID':<5} {'Name':<20} {'Classification':<15} {'Habitat':<20} {'Lifespan':<10}")
    print("-"*80)
    
    for species in species_list:
        print(f"{species[0]:<5} {species[1]:<20} {species[2]:<15} {species[3]:<20} {species[4]:<10}")
    
    print("="*80 + "\n")


def display_species_details(species):
    """Display detailed information about a species"""
    if not species:
        print("Species not found")
        return
    
    print("\n" + "="*80)
    print(f"ID: {species[0]}")
    print(f"Name: {species[1]}")
    print(f"Classification: {species[2]}")
    print(f"Habitat: {species[3]}")
    print(f"Diet: {species[4]}")
    print(f"Lifespan: {species[5]} years")
    print(f"Description: {species[6]}")
    print("="*80 + "\n")


def add_sample_data(db):
    """Add sample data to the database"""
    sample_species = [
        ("African Elephant", "Mammal", "Savannas, forests, deserts", "Herbivore", 70,
         "The African elephant is the largest living terrestrial animal with large ears and tusks."),
        ("Peregrine Falcon", "Bird", "Mountains, coastal areas, urban areas", "Carnivore", 15,
         "The peregrine falcon is renowned for its speed, reaching over 320 km/h during hunting dives."),
        ("Komodo Dragon", "Reptile", "Indonesian islands", "Carnivore", 30,
         "The Komodo dragon is the largest living species of lizard, growing to a maximum length of 3 meters.")
    ]
    
    for species in sample_species:
        db.add_species(*species)


def main_menu():
    """Display the main menu options"""
    print("\n===== ANIMAL SPECIES DATABASE =====")
    print("1. Add a new species")
    print("2. Search for species")
    print("3. View all species")
    print("4. View species details")
    print("5. Update a species")
    print("6. Delete a species")
    print("7. Exit")
    return input("Enter your choice (1-7): ")


def get_species_input(existing_data=None):
    """Get species information from user input"""
    print("\nEnter species information:")
    
    if existing_data:
        name = input(f"Name ({existing_data[1]}): ") or existing_data[1]
        
        print(f"Current classification: {existing_data[2]}")
        print("Available classifications: Mammal, Bird, Reptile, Amphibian, Fish, Invertebrate")
        classification = input("Classification: ") or existing_data[2]
        
        habitat = input(f"Habitat ({existing_data[3]}): ") or existing_data[3]
        diet = input(f"Diet ({existing_data[4]}): ") or existing_data[4]
        
        try:
            lifespan = int(input(f"Average lifespan in years ({existing_data[5]}): ") or existing_data[5])
        except ValueError:
            lifespan = existing_data[5]
            
        description = input(f"Description ({existing_data[6]}): ") or existing_data[6]
    else:
        name = input("Name: ")
        
        print("Available classifications: Mammal, Bird, Reptile, Amphibian, Fish, Invertebrate")
        classification = input("Classification: ")
        
        habitat = input("Habitat: ")
        diet = input("Diet: ")
        
        try:
            lifespan = int(input("Average lifespan in years: "))
        except ValueError:
            print("Invalid input for lifespan. Setting to 0.")
            lifespan = 0
            
        description = input("Description: ")
    
    return name, classification, habitat, diet, lifespan, description


def main():
    """Main function to run the application"""
    db = AnimalSpeciesDatabase()
    
    # Check if the database is empty and add sample data if it is
    if not db.get_all_species():
        add_sample_data(db)
    
    while True:
        choice = main_menu()
        
        if choice == '1':
            # Add a new species
            species_data = get_species_input()
            db.add_species(*species_data)
            
        elif choice == '2':
            # Search for species
            search_term = input("\nEnter search term (name or description): ")
            print("Available classifications: All, Mammal, Bird, Reptile, Amphibian, Fish, Invertebrate")
            classification = input("Filter by classification (or 'All' for no filter): ")
            
            if classification.lower() == 'all':
                classification = None
                
            results = db.search_species(search_term, classification)
            display_species(results)
            
        elif choice == '3':
            # View all species
            all_species = db.get_all_species()
            display_species(all_species)
            
        elif choice == '4':
            # View species details
            species_id = input("\nEnter species ID: ")
            try:
                species_id = int(species_id)
                species = db.get_species_by_id(species_id)
                display_species_details(species)
            except ValueError:
                print("Invalid ID. Please enter a number.")
            
        elif choice == '5':
            # Update a species
            species_id = input("\nEnter ID of species to update: ")
            try:
                species_id = int(species_id)
                species = db.get_species_by_id(species_id)
                
                if species:
                    print(f"Updating species: {species[1]}")
                    updated_data = get_species_input(species)
                    db.update_species(species_id, *updated_data)
                else:
                    print(f"No species found with ID {species_id}")
            except ValueError:
                print("Invalid ID. Please enter a number.")
            
        elif choice == '6':
            # Delete a species
            species_id = input("\nEnter ID of species to delete: ")
            try:
                species_id = int(species_id)
                species = db.get_species_by_id(species_id)
                
                if species:
                    confirm = input(f"Are you sure you want to delete '{species[1]}'? (y/n): ")
                    if confirm.lower() == 'y':
                        db.delete_species(species_id)
                else:
                    print(f"No species found with ID {species_id}")
            except ValueError:
                print("Invalid ID. Please enter a number.")
            
        elif choice == '7':
            # Exit the program
            print("\nThank you for using the Animal Species Database!")
            db.close_connection()
            break
            
        else:
            print("Invalid choice. Please try again.")


if __name__ == "__main__":
    main()
