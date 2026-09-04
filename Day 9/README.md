## Code Node Script

The Code node runs once for all input records. It normalizes names, converts emails to lowercase, calculates a grade based on the score, and filters out records below grade B.

```javascript

// Sample messy dataset
const records = [
  { name: "  john smith ", email: "JOHN.SMITH@GMAIL.COM", score: 92 },
  { name: "sArAh khAn", email: "SARAH.KHAN@GMAIL.COM", score: 85 },
  { name: "  ahmed ali", email: "Ahmed.Ali@GMAIL.COM", score: 78 },
  { name: "FATIMA NOOR ", email: "FATIMA.NOOR@GMAIL.COM", score: 91 },
  { name: "hassan raza", email: "HASSAN.RAZA@GMAIL.COM", score: 73 },
  { name: "  Ayesha Malik", email: "AYESHA.MALIK@GMAIL.COM", score: 88 },
  { name: "MUHAMMAD USMAN", email: "Muhammad.Usman@GMAIL.COM", score: 64 },
  { name: "ali hassan ", email: "ALI.HASSAN@GMAIL.COM", score: 55 },
  { name: "zainab iqbal", email: "ZAINAB.IQBAL@GMAIL.COM", score: 82 },
  { name: "  Bilal Ahmed", email: "BILAL.AHMED@GMAIL.COM", score: 69 },
  { name: "usman khan", email: "USMAN.KHAN@GMAIL.COM", score: 94 },
  { name: "  maryam sheikh ", email: "MARYAM.SHEIKH@GMAIL.COM", score: 76 },
  { name: "HAMZA FAROOQ", email: "HAMZA.FAROOQ@GMAIL.COM", score: 87 },
  { name: "sana javed", email: "SANA.JAVED@GMAIL.COM", score: 59 },
  { name: "  Omar Hassan", email: "OMAR.HASSAN@GMAIL.COM", score: 81 }
];

// Normalize names and emails and calculate grades
const processedRecords = records.map(record => {

  const normalizedName = record.name
    .trim()
    .toLowerCase()
    .split(" ")
    .filter(word => word.length > 0)
    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
    .join(" ");

  let grade;

  if (record.score >= 90) {
    grade = "A";
  } else if (record.score >= 80) {
    grade = "B";
  } else if (record.score >= 70) {
    grade = "C";
  } else if (record.score >= 60) {
    grade = "D";
  } else {
    grade = "F";
  }

  return {
    json: {
      name: normalizedName,
      email: record.email.trim().toLowerCase(),
      score: record.score,
      grade: grade
    }
  };
});

// Keep only records with grade A or B
const filteredRecords = processedRecords.filter(
  record => record.json.grade === "A" || record.json.grade === "B"
);

return filteredRecords;
