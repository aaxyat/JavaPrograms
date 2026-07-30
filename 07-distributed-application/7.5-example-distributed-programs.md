# Unit 7: Distributed Application

## Example Distributed Programs

This section presents **3 complete, real-world RMI distributed systems**. Each example includes complete server and client Java source code followed by an in-depth, line-by-line code explanation.

---

## 1. Distributed Hospital & Patient Records Management System

### Source Code

#### `PatientRecord.java` (Serializable DTO)
```java
import java.io.Serializable;

public class PatientRecord implements Serializable {
    private static final long serialVersionUID = 7001L;

    private String patientId;
    private String name;
    private String diagnosis;
    private boolean isAdmitted;

    public PatientRecord(String patientId, String name, String diagnosis, boolean isAdmitted) {
        this.patientId = patientId;
        this.name = name;
        this.diagnosis = diagnosis;
        this.isAdmitted = isAdmitted;
    }

    public String getPatientId() { return patientId; }
    public String getName() { return name; }
    public String getDiagnosis() { return diagnosis; }
    public boolean isAdmitted() { return isAdmitted; }

    public void setDiagnosis(String diagnosis) { this.diagnosis = diagnosis; }
    public void setAdmitted(boolean admitted) { isAdmitted = admitted; }

    @Override
    public String toString() {
        return String.format("PatientRecord [ID: %s | Name: %-15s | Diagnosis: %-20s | Status: %s]",
                patientId, name, diagnosis, (isAdmitted ? "ADMITTED" : "DISCHARGED"));
    }
}
```

#### `HospitalService.java` (Remote Interface)
```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface HospitalService extends Remote {
    boolean registerPatient(PatientRecord record) throws RemoteException;
    PatientRecord getPatient(String patientId) throws RemoteException;
    boolean updateDiagnosis(String patientId, String newDiagnosis) throws RemoteException;
    boolean dischargePatient(String patientId) throws RemoteException;
}
```

#### `HospitalServiceImpl.java` (Remote Object Implementation)
```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class HospitalServiceImpl extends UnicastRemoteObject implements HospitalService {

    private final Map<String, PatientRecord> patientDatabase = new ConcurrentHashMap<>();

    public HospitalServiceImpl() throws RemoteException {
        super();
        // Seed database
        patientDatabase.put("P-101", new PatientRecord("P-101", "John Doe", "Acute Appendicitis", true));
        patientDatabase.put("P-102", new PatientRecord("P-102", "Jane Smith", "Fractured Radius", false));
    }

    @Override
    public synchronized boolean registerPatient(PatientRecord record) throws RemoteException {
        System.out.println("[HOSPITAL SERVER] Registering new patient: " + record.getName());
        if (patientDatabase.containsKey(record.getPatientId())) return false;
        patientDatabase.put(record.getPatientId(), record);
        return true;
    }

    @Override
    public synchronized PatientRecord getPatient(String patientId) throws RemoteException {
        System.out.println("[HOSPITAL SERVER] Fetching record for ID: " + patientId);
        return patientDatabase.get(patientId);
    }

    @Override
    public synchronized boolean updateDiagnosis(String patientId, String newDiagnosis) throws RemoteException {
        System.out.println("[HOSPITAL SERVER] Updating diagnosis for ID: " + patientId);
        PatientRecord record = patientDatabase.get(patientId);
        if (record != null) {
            record.setDiagnosis(newDiagnosis);
            return true;
        }
        return false;
    }

    @Override
    public synchronized boolean dischargePatient(String patientId) throws RemoteException {
        System.out.println("[HOSPITAL SERVER] Discharging patient ID: " + patientId);
        PatientRecord record = patientDatabase.get(patientId);
        if (record != null) {
            record.setAdmitted(false);
            return true;
        }
        return false;
    }
}
```

#### `HospitalServer.java` (RMI Server Application)
```java
import java.rmi.registry.LocateRegistry;
import java.rmi.Naming;

public class HospitalServer {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("  DISTRIBUTED HOSPITAL MANAGEMENT SERVER ");
        System.out.println("==========================================");

        try {
            LocateRegistry.createRegistry(1099);
            HospitalService service = new HospitalServiceImpl();
            Naming.rebind("rmi://localhost:1099/HospitalService", service);

            System.out.println("HospitalService registered on port 1099.");
            System.out.println("Server ready to process medical records...");

        } catch (Exception e) {
            System.err.println("Hospital Server Error: " + e.getMessage());
        }
    }
}
```

#### `HospitalClient.java` (RMI Client Application)
```java
import java.rmi.Naming;

public class HospitalClient {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("   DISTRIBUTED HOSPITAL MEDICAL CLIENT    ");
        System.out.println("==========================================");

        try {
            HospitalService hospital = (HospitalService) Naming.lookup("rmi://localhost:1099/HospitalService");

            // 1. Fetch Patient Record
            PatientRecord p101 = hospital.getPatient("P-101");
            System.out.println("Retrieved Record: " + p101);

            // 2. Register New Patient
            PatientRecord p103 = new PatientRecord("P-103", "Robert Chen", "Hypertension", true);
            boolean registered = hospital.registerPatient(p103);
            System.out.println("Registered P-103 Success? : " + registered);

            // 3. Update Diagnosis
            boolean updated = hospital.updateDiagnosis("P-101", "Post-Op Appendectomy Recovery");
            System.out.println("Updated P-101 Diagnosis?  : " + updated);

            // 4. Discharge Patient
            boolean discharged = hospital.dischargePatient("P-101");
            System.out.println("Discharged P-101 Success? : " + discharged);

            // Print Final State
            System.out.println("\nFinal State for P-101: " + hospital.getPatient("P-101"));
            System.out.println("==========================================");

        } catch (Exception e) {
            System.err.println("Hospital Client Error: " + e.getMessage());
        }
    }
}
```

### Explanation

- **`PatientRecord.java` Line 1–28**: Implements `Serializable` so patient data objects transfer over RMI socket streams.
- **`HospitalService.java` Line 1–11**: Remote interface extending `Remote`, declaring `throws RemoteException` for medical operations.
- **`HospitalServiceImpl.java` Line 1–45**: Extends `UnicastRemoteObject`, using a thread-safe `ConcurrentHashMap` for patient storage and `synchronized` methods for data consistency.
- **`HospitalServer.java` Line 1–24**: Launches the RMI registry on port 1099 and registers `"HospitalService"`.
- **`HospitalClient.java` Line 1–35**: Looks up the service stub via `Naming.lookup()`, executing remote patient registrations, updates, and discharges.

---

## 2. Distributed E-Learning Exam & Grading System

### Source Code

#### `ExamPaper.java` & `ExamResult.java` (Serializable DTOs)
```java
import java.io.Serializable;

public class ExamPaper implements Serializable {
    private static final long serialVersionUID = 7002L;

    private String examId;
    private String subject;
    private String[] questions;

    public ExamPaper(String examId, String subject, String[] questions) {
        this.examId = examId;
        this.subject = subject;
        this.questions = questions;
    }

    public String getExamId() { return examId; }
    public String getSubject() { return subject; }
    public String[] getQuestions() { return questions; }
}

class ExamSubmission implements Serializable {
    private static final long serialVersionUID = 7003L;

    private String studentId;
    private String examId;
    private String[] answers;

    public ExamSubmission(String studentId, String examId, String[] answers) {
        this.studentId = studentId;
        this.examId = examId;
        this.answers = answers;
    }

    public String getStudentId() { return studentId; }
    public String getExamId() { return examId; }
    public String[] getAnswers() { return answers; }
}

class ExamResult implements Serializable {
    private static final long serialVersionUID = 7004L;

    private String studentId;
    private String subject;
    private double scorePercentage;
    private String grade;

    public ExamResult(String studentId, String subject, double scorePercentage, String grade) {
        this.studentId = studentId;
        this.subject = subject;
        this.scorePercentage = scorePercentage;
        this.grade = grade;
    }

    @Override
    public String toString() {
        return String.format("ExamResult [Student: %s | Subject: %-12s | Score: %.2f%% | Grade: %s]",
                studentId, subject, scorePercentage, grade);
    }
}
```

#### `ExamService.java` (Remote Interface)
```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface ExamService extends Remote {
    ExamPaper getExamPaper(String examId) throws RemoteException;
    ExamResult submitExam(ExamSubmission submission) throws RemoteException;
}
```

#### `ExamServiceImpl.java` (Remote Object Implementation)
```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.HashMap;
import java.util.Map;

public class ExamServiceImpl extends UnicastRemoteObject implements ExamService {

    private final Map<String, String[]> answerKeys = new HashMap<>();

    public ExamServiceImpl() throws RemoteException {
        super();
        // Seed answer key for JAVA-101: Q1="B", Q2="A", Q3="C"
        answerKeys.put("JAVA-101", new String[]{"B", "A", "C"});
    }

    @Override
    public ExamPaper getExamPaper(String examId) throws RemoteException {
        System.out.println("[EXAM SERVER] Dispensing paper for: " + examId);
        String[] questions = {
            "Q1: What is JVM? (A) Compiler (B) Virtual Machine (C) Editor",
            "Q2: Default value of boolean? (A) false (B) true (C) null",
            "Q3: Which is a marker interface? (A) Runnable (B) Cloneable (C) Serializable"
        };
        return new ExamPaper(examId, "Java Computer Science", questions);
    }

    @Override
    public ExamResult submitExam(ExamSubmission submission) throws RemoteException {
        System.out.println("[EXAM SERVER] Grading submission for student: " + submission.getStudentId());
        
        String[] correctAnswers = answerKeys.get(submission.getExamId());
        String[] studentAnswers = submission.getAnswers();

        int correctCount = 0;
        for (int i = 0; i < correctAnswers.length && i < studentAnswers.length; i++) {
            if (correctAnswers[i].equalsIgnoreCase(studentAnswers[i])) {
                correctCount++;
            }
        }

        double score = (correctCount / (double) correctAnswers.length) * 100.0;
        String grade = score >= 90 ? "A+" : (score >= 60 ? "Pass" : "Fail");

        return new ExamResult(submission.getStudentId(), "Java CS", score, grade);
    }
}
```

#### `ExamServer.java` (RMI Server Application)
```java
import java.rmi.registry.LocateRegistry;
import java.rmi.Naming;

public class ExamServer {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("    E-LEARNING EXAM EVALUATION SERVER     ");
        System.out.println("==========================================");

        try {
            LocateRegistry.createRegistry(1099);
            ExamService service = new ExamServiceImpl();
            Naming.rebind("rmi://localhost:1099/ExamService", service);

            System.out.println("ExamService active on port 1099.");
            System.out.println("Server ready to process online exams...");

        } catch (Exception e) {
            System.err.println("Exam Server Error: " + e.getMessage());
        }
    }
}
```

#### `ExamClient.java` (RMI Client Application)
```java
import java.rmi.Naming;

public class ExamClient {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("     E-LEARNING STUDENT EXAM CLIENT       ");
        System.out.println("==========================================");

        try {
            ExamService examService = (ExamService) Naming.lookup("rmi://localhost:1099/ExamService");

            // 1. Fetch Exam Paper
            ExamPaper paper = examService.getExamPaper("JAVA-101");
            System.out.println("Fetched Subject: " + paper.getSubject());
            System.out.println("Displaying Exam Questions:");
            for (String q : paper.getQuestions()) {
                System.out.println("  -> " + q);
            }

            // 2. Submit Answers
            String[] myAnswers = {"B", "A", "C"}; // 100% correct answers
            ExamSubmission submission = new ExamSubmission("STU-9940", "JAVA-101", myAnswers);
            
            System.out.println("\nSubmitting answers to remote grading engine...");
            ExamResult result = examService.submitExam(submission);

            // 3. Display Graded Result
            System.out.println("\nGrading Result Returned:");
            System.out.println(result);
            System.out.println("==========================================");

        } catch (Exception e) {
            System.err.println("Exam Client Error: " + e.getMessage());
        }
    }
}
```

### Explanation

- **DTO Encapsulation (`ExamPaper`, `ExamSubmission`, `ExamResult`)**: Transfers complete exam data bundles across the network in single RMI calls, reducing network round-trips.
- **Server Automated Grading (`ExamServiceImpl.submitExam`)**: Compares student answers against server-side answer keys, calculates percentages, assigns letter grades, and returns a serializable `ExamResult`.

---

## 3. Distributed Flight Reservation & Ticketing Portal

### Source Code

#### `FlightInfo.java` & `BookingReceipt.java` (Serializable DTOs)
```java
import java.io.Serializable;

public class FlightInfo implements Serializable {
    private static final long serialVersionUID = 7005L;

    private String flightNumber;
    private String origin;
    private String destination;
    private double ticketPrice;
    private int availableSeats;

    public FlightInfo(String flightNumber, String origin, String destination, double ticketPrice, int availableSeats) {
        this.flightNumber = flightNumber;
        this.origin = origin;
        this.destination = destination;
        this.ticketPrice = ticketPrice;
        this.availableSeats = availableSeats;
    }

    public String getFlightNumber() { return flightNumber; }
    public String getOrigin() { return origin; }
    public String getDestination() { return destination; }
    public double getTicketPrice() { return ticketPrice; }
    public int getAvailableSeats() { return availableSeats; }
    public void setAvailableSeats(int seats) { availableSeats = seats; }

    @Override
    public String toString() {
        return String.format("Flight %s [%s -> %s] Price: $%.2f | Seats Left: %d",
                flightNumber, origin, destination, ticketPrice, availableSeats);
    }
}

class BookingReceipt implements Serializable {
    private static final long serialVersionUID = 7006L;

    private String bookingRef;
    private String flightNumber;
    private String passengerName;
    private double amountPaid;

    public BookingReceipt(String bookingRef, String flightNumber, String passengerName, double amountPaid) {
        this.bookingRef = bookingRef;
        this.flightNumber = flightNumber;
        this.passengerName = passengerName;
        this.amountPaid = amountPaid;
    }

    @Override
    public String toString() {
        return String.format("Receipt [Ref: %s | Flight: %s | Passenger: %s | Paid: $%.2f]",
                bookingRef, flightNumber, passengerName, amountPaid);
    }
}
```

#### `FlightReservationService.java` (Remote Interface)
```java
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;

public interface FlightReservationService extends Remote {
    List<FlightInfo> searchFlights(String origin, String destination) throws RemoteException;
    BookingReceipt bookFlight(String flightNumber, String passengerName) throws RemoteException;
}
```

#### `FlightReservationServiceImpl.java` (Remote Object Implementation)
```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.List;
import java.util.ArrayList;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.UUID;

public class FlightReservationServiceImpl extends UnicastRemoteObject implements FlightReservationService {

    private final Map<String, FlightInfo> flightCatalog = new ConcurrentHashMap<>();

    public FlightReservationServiceImpl() throws RemoteException {
        super();
        flightCatalog.put("FL-101", new FlightInfo("FL-101", "KTM", "DEL", 250.00, 5));
        flightCatalog.put("FL-202", new FlightInfo("FL-202", "KTM", "BKK", 420.00, 2));
    }

    @Override
    public synchronized List<FlightInfo> searchFlights(String origin, String destination) throws RemoteException {
        System.out.println("[FLIGHT SERVER] Searching flights for " + origin + " -> " + destination);
        List<FlightInfo> results = new ArrayList<>();
        for (FlightInfo flight : flightCatalog.values()) {
            if (flight.getOrigin().equalsIgnoreCase(origin) && flight.getDestination().equalsIgnoreCase(destination)) {
                results.add(flight);
            }
        }
        return results;
    }

    @Override
    public synchronized BookingReceipt bookFlight(String flightNumber, String passengerName) throws RemoteException {
        System.out.println("[FLIGHT SERVER] Booking request for " + flightNumber + " by " + passengerName);
        FlightInfo flight = flightCatalog.get(flightNumber);

        if (flight != null && flight.getAvailableSeats() > 0) {
            flight.setAvailableSeats(flight.getAvailableSeats() - 1); // Decrement seat count
            String ref = "PNR-" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();
            return new BookingReceipt(ref, flightNumber, passengerName, flight.getTicketPrice());
        }
        return null; // Booking failed (No seats or invalid flight)
    }
}
```

#### `FlightServer.java` (RMI Server Application)
```java
import java.rmi.registry.LocateRegistry;
import java.rmi.Naming;

public class FlightServer {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("  DISTRIBUTED FLIGHT RESERVATION SERVER   ");
        System.out.println("==========================================");

        try {
            LocateRegistry.createRegistry(1099);
            FlightReservationService service = new FlightReservationServiceImpl();
            Naming.rebind("rmi://localhost:1099/FlightReservationService", service);

            System.out.println("FlightReservationService registered on port 1099.");
            System.out.println("Flight Server ready for remote bookings...");

        } catch (Exception e) {
            System.err.println("Flight Server Error: " + e.getMessage());
        }
    }
}
```

#### `FlightClient.java` (RMI Client Application)
```java
import java.rmi.Naming;
import java.util.List;

public class FlightClient {

    public static void main(String[] args) {
        System.out.println("==========================================");
        System.out.println("   DISTRIBUTED FLIGHT BOOKING CLIENT      ");
        System.out.println("==========================================");

        try {
            FlightReservationService flightService = 
                    (FlightReservationService) Naming.lookup("rmi://localhost:1099/FlightReservationService");

            // 1. Search Available Flights
            System.out.println("Searching flights from KTM to DEL...");
            List<FlightInfo> availableFlights = flightService.searchFlights("KTM", "DEL");

            for (FlightInfo f : availableFlights) {
                System.out.println("  Found: " + f);
            }

            // 2. Book Flight Seat
            if (!availableFlights.isEmpty()) {
                FlightInfo target = availableFlights.get(0);
                System.out.println("\nBooking seat on flight " + target.getFlightNumber() + "...");

                BookingReceipt receipt = flightService.bookFlight(target.getFlightNumber(), "Aarav Sharma");

                if (receipt != null) {
                    System.out.println("BOOKING SUCCESS!");
                    System.out.println(receipt);
                } else {
                    System.out.println("BOOKING FAILED: Flight is fully booked.");
                }
            }

            System.out.println("==========================================");

        } catch (Exception e) {
            System.err.println("Flight Client Error: " + e.getMessage());
        }
    }
}
```

### Explanation

- **Thread-Safe Seat Decrement**: `FlightReservationServiceImpl.bookFlight()` uses `synchronized` blocks and `ConcurrentHashMap` to verify available seats and update seat inventories safely when multiple clients attempt concurrent bookings.
- **Dynamic PNR Receipt Generation**: Returns a serializable `BookingReceipt` containing a unique PNR reservation code generated via `UUID`.
