using System;
using System.Collections.Generic;
using System.IO;
using System.Text.Json;

namespace MotelManagementSystem
{
    class Program
    {
        static List<Room> rooms = new List<Room>();
        static string filePath = "motel_data.json";

        static void Main()
        {
            LoadData();
            int choice;

            do
            {
                Console.Clear();
                Console.WriteLine("===== MOTEL MANAGEMENT SYSTEM =====");
                Console.WriteLine("1. View Rooms");
                Console.WriteLine("2. Reserve Room");
                Console.WriteLine("3. Cancel Reservation");
                Console.WriteLine("4. Check-In");
                Console.WriteLine("5. Check-Out & Billing");
                Console.WriteLine("6. Exit");
                choice = GetValidInt("Choose option: ");

                switch (choice)
                {
                    case 1: ViewRooms(); break;
                    case 2: ReserveRoom(); break;
                    case 3: CancelReservation(); break;
                    case 4: CheckIn(); break;
                    case 5: CheckOut(); break;
                }

                SaveData();
                Console.WriteLine("\nPress ENTER to continue...");
                Console.ReadLine();

            } while (choice != 6);
        }

        // ================= FILE HANDLING =================
        static void LoadData()
        {
            if (File.Exists(filePath))
            {
                string json = File.ReadAllText(filePath);
                rooms = JsonSerializer.Deserialize<List<Room>>(json);
            }
            else
            {
                InitializeRooms();
                SaveData();
            }
        }

        static void SaveData()
        {
            string json = JsonSerializer.Serialize(rooms, new JsonSerializerOptions { WriteIndented = true });
            File.WriteAllText(filePath, json);
        }

        // ================= INPUT =================
        static int GetValidInt(string prompt)
        {
            int value;
            while (true)
            {
                Console.Write(prompt);
                if (int.TryParse(Console.ReadLine(), out value))
                    return value;
                Console.WriteLine("Invalid input.");
            }
        }

        static DateTime GetValidDateTime(string prompt)
        {
            DateTime dt;
            while (true)
            {
                Console.Write(prompt);
                if (DateTime.TryParse(Console.ReadLine(), out dt))
                    return dt;
                Console.WriteLine("Invalid date/time.");
            }
        }

        // ================= ROOMS =================
        static void InitializeRooms()
        {
            rooms = new List<Room>
            {
                new Room(101,"Single",1000),
                new Room(102,"Double",1500),
                new Room(103,"Family",2000),
                new Room(104,"Single",1000),
                new Room(105,"Single",1000),
                new Room(201,"Double",1500),
                new Room(202,"Double",1500),
                new Room(203,"Family",2000),
                new Room(301,"Suite",2500),
                new Room(302,"Suite",2500)
            };
        }

        static void ViewRooms()
        {
            Console.WriteLine("\nRoom | Type | Price | Status    | Guest | Reservation");
            Console.WriteLine("------------------------------------------------------");
            foreach (Room r in rooms)
            {
                string res = r.ReservationDate == null ? "N/A" : r.ReservationDate.ToString();
                Console.WriteLine($"{r.RoomNo} | {r.Type,-6} | PHP {r.Price} | {r.Status,-9} | {r.GuestName,-6} | {res}");
            }
        }

        // ================= RESERVATION =================
        static void ReserveRoom()
        {
            int roomNo = GetValidInt("Enter Room Number: ");
            Room room = rooms.Find(r => r.RoomNo == roomNo);

            if (room == null || room.Status != "Available")
            {
                Console.WriteLine("Room unavailable.");
                return;
            }

            Console.Write("Guest Name: ");
            room.GuestName = Console.ReadLine();
            room.BookedDays = GetValidInt("Planned days: ");
            room.ReservationDate = GetValidDateTime("Reservation date & time: ");
            room.Status = "Reserved";

            Console.WriteLine("Reservation successful.");
        }

        static void CancelReservation()
        {
            int roomNo = GetValidInt("Enter Room Number: ");
            Room room = rooms.Find(r => r.RoomNo == roomNo);

            if (room == null || room.Status != "Reserved")
            {
                Console.WriteLine("No reservation found.");
                return;
            }

            room.Reset();
            Console.WriteLine("Reservation cancelled.");
        }

        // ================= CHECK-IN =================
        static void CheckIn()
        {
            int roomNo = GetValidInt("Enter Room Number: ");
            Room room = rooms.Find(r => r.RoomNo == roomNo);

            if (room == null || room.Status == "Occupied")
            {
                Console.WriteLine("Room unavailable.");
                return;
            }

            if (room.Status == "Available")
            {
                Console.Write("Guest Name: ");
                room.GuestName = Console.ReadLine();
                room.BookedDays = GetValidInt("Days to stay: ");
            }

            Console.Write("Breakfast? (y/n): ");
            if (Console.ReadLine().ToLower() == "y") room.Breakfast = ChooseBreakfast();
            Console.Write("Lunch? (y/n): ");
            if (Console.ReadLine().ToLower() == "y") room.Lunch = ChooseLunch();
            Console.Write("Dinner? (y/n): ");
            if (Console.ReadLine().ToLower() == "y") room.Dinner = ChooseDinner();

            room.Status = "Occupied";
            Console.WriteLine("Checked in successfully.");
        }

        // ================= MENUS =================
        static Meal ChooseBreakfast()
        {
            Console.WriteLine("1. Tapsilog PHP 150");
            Console.WriteLine("2. Pancakes PHP 120");
            Console.WriteLine("3. Continental PHP 180");
            int c = GetValidInt("Choose: ");
            return c == 1 ? new Meal("Tapsilog",150) :
                   c == 2 ? new Meal("Pancakes",120) :
                            new Meal("Continental",180);
        }

        static Meal ChooseLunch()
        {
            Console.WriteLine("1. Chicken Adobo PHP 250");
            Console.WriteLine("2. Beef Steak PHP 300");
            Console.WriteLine("3. Pasta PHP 220");
            int c = GetValidInt("Choose: ");
            return c == 1 ? new Meal("Chicken Adobo",250) :
                   c == 2 ? new Meal("Beef Steak",300) :
                            new Meal("Pasta",220);
        }

        static Meal ChooseDinner()
        {
            Console.WriteLine("1. Grilled Fish PHP 280");
            Console.WriteLine("2. Pork Chop PHP 320");
            Console.WriteLine("3. Rice Bowl PHP 200");
            int c = GetValidInt("Choose: ");
            return c == 1 ? new Meal("Grilled Fish",280) :
                   c == 2 ? new Meal("Pork Chop",320) :
                            new Meal("Rice Bowl",200);
        }

        // ================= CHECK-OUT =================
        static void CheckOut()
        {
            int roomNo = GetValidInt("Enter Room Number: ");
            Room room = rooms.Find(r => r.RoomNo == roomNo);

            if (room == null || room.Status != "Occupied")
            {
                Console.WriteLine("Invalid room.");
                return;
            }

            int actualDays = GetValidInt("Actual days stayed: ");
            int exceed = Math.Max(0, actualDays - room.BookedDays);

            double roomCharge = room.BookedDays * room.Price;
            double penalty = exceed * (room.Price * 1.5);

            double mealTotal = 0;
            if (room.Breakfast != null) mealTotal += room.Breakfast.Price * room.BookedDays;
            if (room.Lunch != null) mealTotal += room.Lunch.Price * room.BookedDays;
            if (room.Dinner != null) mealTotal += room.Dinner.Price * room.BookedDays;

            double discount = (exceed == 0 && actualDays >= 5) ? roomCharge * 0.05 : 0;
            double total = roomCharge + mealTotal + penalty - discount;

            Console.WriteLine("\n===== RECEIPT =====");
            Console.WriteLine($"Guest : {room.GuestName}");
            Console.WriteLine($"Room  : {room.RoomNo}");
            Console.WriteLine($"Room Charge : PHP {roomCharge}");
            Console.WriteLine($"Meal Total  : PHP {mealTotal}");
            Console.WriteLine($"Penalty     : PHP {penalty}");
            Console.WriteLine($"Discount    : PHP {discount}");
            Console.WriteLine($"TOTAL BILL  : PHP {total}");
            Console.WriteLine("===================");

            room.Reset();
        }
    }

    // ================= CLASSES =================
    class Room
    {
        public int RoomNo { get; set; }
        public string Type { get; set; }
        public double Price { get; set; }
        public string Status { get; set; } = "Available";
        public string GuestName { get; set; } = "None";
        public int BookedDays { get; set; }
        public DateTime? ReservationDate { get; set; }
        public Meal Breakfast { get; set; }
        public Meal Lunch { get; set; }
        public Meal Dinner { get; set; }

        public Room() { }
        public Room(int no, string type, double price)
        {
            RoomNo = no;
            Type = type;
            Price = price;
        }

        public void Reset()
        {
            Status = "Available";
            GuestName = "None";
            BookedDays = 0;
            ReservationDate = null;
            Breakfast = Lunch = Dinner = null;
        }
    }

    class Meal
    {
        public string Name { get; set; }
        public double Price { get; set; }

        public Meal() { }
        public Meal(string name, double price)
        {
            Name = name;
            Price = price;
        }
    }
}
