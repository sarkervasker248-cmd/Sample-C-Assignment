# Sample-C-Assignment
My first C# work

using System;

// ENUM
enum Semester
{
    Spring = 1,
    Summer = 2,
    Fall = 3
}

// STRUCT
struct MarkSheet
{
    public int Attendance;
    public int ClassTest;
    public int MidTerm;
    public int FinalTerm;

    public int Total()
    {
        return Attendance + ClassTest + MidTerm + FinalTerm;
    }
}

// COURSE CLASS
class Course
{
    public readonly string Code;
    public string Title { get; set; }
    public int Credit { get; set; }

    public Course()
    {
        Code = "CSC 0000";
        Title = "Untitled";
        Credit = 3;
    }

    public Course(string code, string title, int credit)
    {
        Code = code;
        Title = title;
        Credit = credit;
    }

    // Copy Constructor
    public Course(Course oldCourse)
    {
        Code = oldCourse.Code;
        Title = oldCourse.Title;
        Credit = oldCourse.Credit;
    }

    public override string ToString()
    {
        return Code + " " + Title + ", " + Credit + " credits";
    }
}

// BASE STUDENT CLASS
abstract class Student
{
    private readonly string id;
    private string name;
    private string pin;

    public string ID
    {
        get { return id; }
    }

    public string Name
    {
        get { return name; }
        set
        {
            if (value != "")
                name = value;
            else
                Console.WriteLine("Name cannot be empty");
        }
    }

    public string Department { get; set; }

    public double? CGPA { get; private set; }

    public static int TotalStudents = 0;

    public const int MAX_MARKS = 100;

    // Single dimensional array
    protected double[] semesterGPA = new double[3];

    // 2D array
    protected int[,] semesterTable = new int[3, 2];

    // Jagged arrays
    protected Course[][] courses = new Course[3][];
    protected MarkSheet[][] marks = new MarkSheet[3][];

    // Static constructor
    static Student()
    {
        Console.WriteLine("Student class initialized");
    }

    public Student(string id, string name, string department)
    {
        this.id = id;
        this.name = name;
        Department = department;

        for (int i = 0; i < 3; i++)
        {
            courses[i] = new Course[0];
            marks[i] = new MarkSheet[0];
        }

        TotalStudents++;
    }

    // Set PIN
    public void SetPIN(string newPin)
    {
        int number;

        if (newPin.Length == 4 &&
            int.TryParse(newPin, out number))
        {
            pin = newPin;
        }
        else
        {
            Console.WriteLine("PIN must have 4 digits");
        }
    }

    // Check PIN
    public bool CheckPIN(string enteredPin)
    {
        return pin == enteredPin;
    }

    // Add result
    public void AddResult(
        Semester semester, Course course, MarkSheet mark)
    {
        int index = (int)semester - 1;

        Array.Resize(ref courses[index],
            courses[index].Length + 1);

        Array.Resize(ref marks[index],
            marks[index].Length + 1);

        // Course reference is shared
        courses[index][courses[index].Length - 1] = course;

        // Struct value is copied
        marks[index][marks[index].Length - 1] = mark;

        CalculateResults();
    }

    // Grace marks
    public int ApplyGrace(int total)
    {
        if (total >= 48 && total < 50)
            return 50;

        return total;
    }

    // Grade
    public string GetGrade(int total)
    {
        total = ApplyGrace(total);

        if (total >= 90) return "A+";
        else if (total >= 85) return "A";
        else if (total >= 80) return "B+";
        else if (total >= 75) return "B";
        else if (total >= 70) return "C+";
        else if (total >= 65) return "C";
        else if (total >= 60) return "D+";
        else if (total >= 50) return "D";
        else return "F";
    }

    // Grade Point
    public double GetGradePoint(int total)
    {
        total = ApplyGrace(total);

        if (total >= 90) return 4.00;
        else if (total >= 85) return 3.75;
        else if (total >= 80) return 3.50;
        else if (total >= 75) return 3.25;
        else if (total >= 70) return 3.00;
        else if (total >= 65) return 2.75;
        else if (total >= 60) return 2.50;
        else if (total >= 50) return 2.25;
        else return 0.00;
    }

    // Calculate GPA and CGPA
    public void CalculateResults()
    {
        double allPoints = 0;
        int allCredits = 0;

        for (int i = 0; i < 3; i++)
        {
            int attempted = 0;
            int earned = 0;
            double points = 0;

            for (int j = 0; j < courses[i].Length; j++)
            {
                int total = marks[i][j].Total();
                int credit = courses[i][j].Credit;

                double gp = GetGradePoint(total);

                attempted += credit;
                points += gp * credit;

                if (GetGrade(total) != "F")
                    earned += credit;
            }

            semesterTable[i, 0] = attempted;
            semesterTable[i, 1] = earned;

            if (attempted > 0)
                semesterGPA[i] =
                    Math.Round(points / attempted, 2);
            else
                semesterGPA[i] = 0;

            allPoints += points;
            allCredits += attempted;
        }

        if (allCredits > 0)
            CGPA = Math.Round(allPoints / allCredits, 2);
        else
            CGPA = null;
    }

    // Earned credits
    public int EarnedCredits()
    {
        int total = 0;

        for (int i = 0; i < 3; i++)
            total += semesterTable[i, 1];

        return total;
    }

    // Get semester GPA
    public double GetSemesterGPA(int semester)
    {
        return semesterGPA[semester - 1];
    }

    // Clear semester GPA
    public void ClearSemesterGPA(int semester)
    {
        semesterGPA[semester - 1] = 0;
    }

    // Student details
    public void PrintDetails()
    {
        Console.WriteLine(
            ID + " | " +
            Name + " | " +
            Department.ToUpper());

        PrintTypeDetails();
    }

    // Abstract methods
    public abstract void PrintTypeDetails();
    public abstract int RequiredCredits();
    public abstract bool IsEligible();
    public abstract string StudentType();

    // Print transcript
    public void PrintTranscript()
    {
        string[] names = { "Spring", "Summer", "Fall" };

        for (int i = 0; i < 3; i++)
        {
            Console.WriteLine(names[i]);

            if (courses[i].Length == 0)
            {
                Console.WriteLine(" No courses");
                continue;
            }

            for (int j = 0; j < courses[i].Length; j++)
            {
                int total = marks[i][j].Total();

                Console.WriteLine(
                    " " + courses[i][j].Code + " " +
                    courses[i][j].Title + " " +
                    courses[i][j].Credit + " " +
                    ApplyGrace(total) + " " +
                    GetGrade(total) + " " +
                    GetGradePoint(total));
            }
        }

        Console.WriteLine(
            "Semester Attempted Earned GPA");

        for (int i = 0; i < 3; i++)
        {
            Console.WriteLine(
                names[i] + "    " +
                semesterTable[i, 0] + "         " +
                semesterTable[i, 1] + "     " +
                semesterGPA[i].ToString("F2"));
        }

        Console.WriteLine(
            "(2D array, rank " + semesterTable.Rank + ")");
    }
}

// UNDERGRADUATE CLASS
class Undergraduate : Student
{
    public Undergraduate(string id, string name, string dept)
        : base(id, name, dept)
    {
    }

    public override int RequiredCredits()
    {
        return 140;
    }

    public override bool IsEligible()
    {
        return CGPA.HasValue &&
               EarnedCredits() >= 140 &&
               CGPA.Value >= 2.00;
    }

    public override string StudentType()
    {
        return "Undergraduate";
    }

    public override void PrintTypeDetails()
    {
        string cgpaText = CGPA.HasValue
            ? CGPA.Value.ToString("F2")
            : "Not available";

        Console.WriteLine(
            " Undergraduate Student, needs 140 credits, CGPA: "
            + cgpaText);
    }
}

// GRADUATE CLASS
class Graduate : Student
{
    public double? ThesisMarks { get; set; }

    public Graduate(string id, string name, string dept)
        : base(id, name, dept)
    {
    }

    public override int RequiredCredits()
    {
        return 36;
    }

    public override bool IsEligible()
    {
        return CGPA.HasValue &&
               EarnedCredits() >= 36 &&
               CGPA.Value >= 3.00 &&
               ThesisMarks.HasValue &&
               ThesisMarks.Value >= 60;
    }

    public override string StudentType()
    {
        return "Graduate";
    }

    public override void PrintTypeDetails()
    {
        string cgpaText = CGPA.HasValue
            ? CGPA.Value.ToString("F2")
            : "Not available";

        Console.WriteLine(
            " Graduate Student, needs 36 credits, CGPA: "
            + cgpaText);
    }
}

// MAIN PROGRAM
class Program
{
    // Value parameter
    static void ChangeValue(int number)
    {
        number = 100;
    }

    // Ref parameter
    static void AddBonus(ref int total, int bonus)
    {
        total += bonus;
    }

    // Out parameter
    static void GetResult(int a, int b, out int result)
    {
        result = a + b;
    }

    // Params parameter
    static void PrintNumbers(params int[] numbers)
    {
        foreach (int n in numbers)
            Console.Write(n + " ");

        Console.WriteLine();
    }

    // ID validation
    static bool ValidID(string id)
    {
        if (id.Length != 10)
            return false;

        string[] parts = id.Split('-');

        if (parts.Length != 3)
            return false;

        if (parts[0].Length != 2 ||
            parts[1].Length != 5 ||
            parts[2].Length != 1)
            return false;

        int year, number;

        if (!int.TryParse(parts[0], out year))
            return false;

        if (!int.TryParse(parts[1], out number))
            return false;

        if (parts[2] != "1" && parts[2] != "2")
            return false;

        return true;
    }

    // Find student
    static Student FindStudent(
        Student[] students, int count, string id)
    {
        for (int i = 0; i < count; i++)
        {
            if (students[i].ID == id)
                return students[i];
        }

        return null;
    }

    // Process result line
    static bool AddResultLine(Student student, string line)
    {
        string[] data = line.Split(',');

        if (data.Length != 8)
            return false;

        int semester = 0;
        int credit = 0;
        int attendance = 0;
        int test = 0;
        int mid = 0;
        int final = 0;

        bool valid =
            int.TryParse(data[0], out semester) &&
            int.TryParse(data[3], out credit) &&
            int.TryParse(data[4], out attendance) &&
            int.TryParse(data[5], out test) &&
            int.TryParse(data[6], out mid) &&
            int.TryParse(data[7], out final);

        if (!valid)
            return false;

        if (semester < 1 || semester > 3 ||
            credit < 1 || credit > 4 ||
            attendance < 0 || attendance > 10 ||
            test < 0 || test > 20 ||
            mid < 0 || mid > 30 ||
            final < 0 || final > 40)
            return false;

        Course course = new Course(
            data[1], data[2], credit);

        MarkSheet mark = new MarkSheet();

        mark.Attendance = attendance;
        mark.ClassTest = test;
        mark.MidTerm = mid;
        mark.FinalTerm = final;

        student.AddResult(
            (Semester)semester, course, mark);

        return true;
    }

    static void Main()
    {
        Console.WriteLine(
            "STUDENT TRANSCRIPT MANAGEMENT SYSTEM");

        // Value parameter
        int value = 10;
        ChangeValue(value);

        // Ref parameter
        int total = 48;
        int original = total;

        AddBonus(ref total, 5);

        // Out parameter
        GetResult(10, 20, out int answer);

        // Params parameter
        PrintNumbers(10, 20, 30);

        // Boxing and unboxing
        int a = 25;
        object obj = a;
        int b = (int)obj;

        // Var and dynamic
        var message = "Welcome to STMS";

        dynamic dynamicValue = 100;
        dynamicValue = "Student System";

        // Type conversion
        string inputNumber = "50";
        int converted = Convert.ToInt32(inputNumber);

        // STUDENT REGISTRATION
        Console.WriteLine("\nSTUDENT REGISTRATION");

        Student[] students = new Student[2];

        // Invalid ID
        string invalidID = "2512-1";

        if (!ValidID(invalidID))
        {
            Console.WriteLine(
                "Registration failed: invalid ID " + invalidID);
        }

        // Valid students
        students[0] = new Undergraduate(
            "25-12345-1",
            "Nusrat Jahan",
            "Computer Science");

        students[1] = new Graduate(
            "26-54321-2",
            "Tanvir Ahmed",
            "Computer Science");

        int count = 2;

        Console.WriteLine(
            "Total students: " + Student.TotalStudents);

        // Name validation
        students[0].Name = "";

        // PIN test
        students[0].SetPIN("1234");
        students[0].SetPIN("12");

        students[1].SetPIN("5678");

        // STUDENT DETAILS
        Console.WriteLine("\nSTUDENT DETAILS");

        for (int i = 0; i < count; i++)
        {
            students[i].PrintDetails();
        }

        // COURSE COPY TEST
        Console.WriteLine("\nCOURSE COPY TEST");

        Course originalCourse = new Course(
            "ENG 1101", "English I", 3);

        Course secondName = originalCourse;

        secondName.Title = "English Basics";

        Course copiedCourse = new Course(originalCourse);

        Console.WriteLine(
            "Course: original " + originalCourse.Title +
            ", alias " + secondName.Title +
            ", copy English I");

        // MARKSHEET COPY TEST
        MarkSheet originalMark = new MarkSheet();

        originalMark.Attendance = 9;
        originalMark.ClassTest = 17;
        originalMark.MidTerm = 24;
        originalMark.FinalTerm = 33;

        MarkSheet copiedMark = originalMark;

        copiedMark.FinalTerm = 40;

        Console.WriteLine(
            "MarkSheet: original " + originalMark.Total() +
            ", copy " + copiedMark.Total());

        // GRACE MARK TEST
        int rawTotal = 48;
        int copiedTotal = rawTotal;

        AddBonus(ref copiedTotal, 5);

        Console.WriteLine(
            "Raw total " + rawTotal +
            ", with 5 bonus " + copiedTotal +
            ", raw total still " + rawTotal);

        Console.WriteLine(
            "After grace " + students[0].ApplyGrace(rawTotal) +
            ", grade " + students[0].GetGrade(rawTotal) +
            ", grade point " +
            students[0].GetGradePoint(rawTotal).ToString("F2"));

        // SAMPLE RESULT DATA
        string[] nusratResults =
        {
            "1,CSC 1101,Introduction to Computing,3,9,17,24,33",
            "1,MAT 1101,Calculus I,3,8,14,20,30",
            "3,PHY 1101,Physics I,3,6,10,15,17",
            "3,CSC 1202,Discrete Mathematics,3,5,8,10,15",
            "3,STA 1101,Statistics,3,9,15,x,29",
            "3,STA 1101,Statistics,3,9,15,21,29",
            "3,ENG 1101,English I,5,10,18,26,36"
        };

        string[] tanvirResults =
        {
            "1,CSC 5101,Advanced Algorithms,3,9,18,25,34",
            "1,CSC 5102,Research Methodology,3,10,19,28,37"
        };

        // RESULT ENTRY
        Console.WriteLine("\nRESULT ENTRY");

        for (int i = 0; i < nusratResults.Length; i++)
        {
            if (!AddResultLine(students[0], nusratResults[i]))
            {
                Console.WriteLine(
                    "Line " + (i + 1) +
                    " skipped: invalid marks or credit hour");
            }
        }

        for (int i = 0; i < tanvirResults.Length; i++)
        {
            AddResultLine(students[1], tanvirResults[i]);
        }

        // PIN VERIFICATION
        Console.WriteLine("\nPIN VERIFICATION");

        Console.WriteLine(
            "PIN 4321: " +
            (students[0].CheckPIN("4321")
            ? "access granted" : "access denied"));

        Console.WriteLine(
            "PIN 1234: " +
            (students[0].CheckPIN("1234")
            ? "access granted" : "access denied"));

        // NUSRAT TRANSCRIPT
        Console.WriteLine("\nNUSRAT TRANSCRIPT");

        students[0].PrintTranscript();

        // SEMESTER GPA AVERAGE
        Console.WriteLine("\nSEMESTER GPA AVERAGE");

        int first = (int)Semester.Spring;
        int second = (int)Semester.Fall;

        double average =
            (students[0].GetSemesterGPA(first) +
             students[0].GetSemesterGPA(second)) / 2;

        Console.WriteLine(
            "Average of semester GPAs: " +
            average.ToString("F2"));

        students[0].ClearSemesterGPA(first);
        students[0].ClearSemesterGPA(second);

        Console.WriteLine(
            "First value after reset: " +
            students[0].GetSemesterGPA(first));

        // FINAL STUDENT SUMMARY
        Console.WriteLine("\nFINAL STUDENT SUMMARY");

        for (int i = 0; i < count; i++)
        {
            Student s = students[i];

            int earned = s.EarnedCredits();

            int remaining = s.RequiredCredits() - earned;

            if (remaining < 0)
                remaining = 0;

            Console.WriteLine("\n" + s.Name + ":");

            Console.WriteLine(s.ID + " (String)");

            Console.WriteLine(earned + " (Int32)");

            Console.WriteLine(
                s.CGPA.Value.ToString("F2") + " (Double)");

            Console.WriteLine(
                s.IsEligible() + " (Boolean)");

            if (s.IsEligible())
                Console.WriteLine("Eligible");
            else
                Console.WriteLine(
                    "Not eligible, " +
                    remaining + " credits remaining");

            if (s is Graduate)
            {
                Graduate g = (Graduate)s;

                Console.WriteLine(
                    "Thesis marks: " +
                    (g.ThesisMarks.HasValue
                    ? g.ThesisMarks.ToString()
                    : "Not available"));
            }

            Console.WriteLine(
                "Graduate student: " +
                (s.StudentType() == "Graduate"));
        }

        // EMPTY COURSE
        Course emptyCourse = new Course();

        Console.WriteLine("\n" + emptyCourse + " (Course)");

        Console.WriteLine("\nProgram finished.");
    }
}
