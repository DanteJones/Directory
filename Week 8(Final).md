using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;
using System.Diagnostics;
using ConsoleApp;

namespace Final_Project
{
class Program
{
    private static List<CrimeData> CrimeDataList = new List<CrimeData>();

static void Main(string[] args)
        {

            string csvFilePath = string.Empty;
            string reportFilePath = string.Empty;

            string startupPath = Directory.GetCurrentDirectory();

            // The Debgger //
            if (Debugger.IsAttached)
            {
                csvFilePath = Path.Combine(startupPath, "CrimeData.csv");
                reportFilePath = Path.Combine(startupPath, "CrimeReport.txt");
            }
            else
            {
                if (args.Length != 2)
                {
                    Console.WriteLine("Invalid call.\n Valid call example : CrimeAnalyzer <crime_csv_file_path> <report_file_path>");
                    Console.ReadLine();
                    return;
                }
                else
                {
                    csvFilePath = args[0];
                    reportFilePath = args[1];

                    if (!csvFilePath.Contains("\\"))
                    {
                        csvFilePath = Path.Combine(startupPath, csvFilePath);
                    }

                    if (!reportFilePath.Contains("\\"))
                    {
                        reportFilePath = Path.Combine(startupPath, reportFilePath);
                    }
                }
            }

            if (File.Exists(csvFilePath))
            {
                if (ReadData(csvFilePath))
                {
                    try
                    {
                        var file = File.Create(reportFilePath);
                        file.Close();
                    }
                    catch (Exception)
                    {
                        Console.WriteLine($"Unable to create report file at : {reportFilePath}");
                    }
                    WriteReport(reportFilePath);
                }
            }
            else
            {
                Console.Write($"Crime data file does not exist at path: {csvFilePath}");
            }

            Console.ReadLine();
        }

        private static bool ReadData(string filePath)
        {
            Console.WriteLine($"Reading data from file : {filePath}");
            try
            {
                int columns = 0;
                string[] crimeDataLines = File.ReadAllLines(filePath);
                for (int index = 0; index < crimeDataLines.Length; index++)
                {
                    string crimeDataLine = crimeDataLines[index];
                    string[] data = crimeDataLine.Split(',');

                    if (index == 0)
                    {
                        columns = data.Length;
                    }
                    else
                    {
                        if (columns != data.Length)
                        {
                            Console.WriteLine($"Row {index} contains {data.Length} values. It should contain {columns}.");
                            return false;
                        }
                        else
                        {
                            try
                            {
                                CrimeData crimeData = new CrimeData
                                {
                                    Year = Convert.ToInt32(data[0]),
                                    Population = Convert.ToInt32(data[1]),
                                    Murders = Convert.ToInt32(data[2]),
                                    Rapes = Convert.ToInt32(data[3]),
                                    Robberies = Convert.ToInt32(data[4]),
                                    ViolentCrimes = Convert.ToInt32(data[5]),
                                    Thefts = Convert.ToInt32(data[6]),
                                    MotorVehicleThefts = Convert.ToInt32(data[7])
                                };

                                CrimeDataList.Add(crimeData);
                            }
                            catch (InvalidCastException)
                            {
                                string value = $"Row {index} contains invalid value.";
                                Console.WriteLine(value);
                                return false;
                            }
                        }
                    }
                }
                Console.WriteLine("Data read completed successfully.");
                return true;
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error in reading data from csv file.");
                throw ex;
            }
        }
        private static void WriteReport(string filePath)
        {
            try
            {
                if (CrimeDataList != null && CrimeDataList.Any())
                {
                    Console.WriteLine($"Calculating the desired data and writing it to report file : {filePath}");

                    StringBuilder sb = new StringBuilder();
                    const string Value = "Crime Analyzer Report \n ";
                    sb.Append(Value);
                    sb.Append(Environment.NewLine);
                    // Question Number 1
                    // Period
                    int minYear = CrimeDataList.Min(x => x.Year);
                    int maxYear = CrimeDataList.Max(x => x.Year);
                    // Years
                    int years = maxYear - minYear + 1;
                    sb.Append(value: $"Period: {minYear}-{maxYear} ({years} years) \n ");
                    sb.Append(Environment.NewLine);

                    // Question Number 2
                    var mYears = from crimeData in CrimeDataList
                                 where crimeData.Murders < 15000
                                 select crimeData.Year;

                    string mYearsStr = string.Empty;
                    for (int i = 0; i < mYears.Count(); i++)
                    {
                        mYearsStr += mYears.ElementAt(i).ToString();
                        if (i < mYears.Count() - 1) mYearsStr += ", ";
                    }
                    sb.Append($"Years murders per year < 15000: {mYearsStr}");
                    sb.Append(Environment.NewLine);

                    // Question Number 3 & 4
                    var rYears = from crimeData in CrimeDataList
                                 where crimeData.Robberies > 500000
                                 select crimeData;

                    string rYearsStr = string.Empty;
                    for (int i = 0; i < rYears.Count(); i++)
                    {
                        CrimeData crimeData = rYears.ElementAt(i);
                        rYearsStr += $"{crimeData.Year} = {crimeData.Robberies}";
                        if (i < rYears.Count() - 1) rYearsStr += ", ";
                    }
                    sb.Append($"Robberies per year > 500000: {rYearsStr}");
                    sb.Append(Environment.NewLine);

                    // Question Number 5
                    var vCrime = from crimeData in CrimeDataList
                                 where crimeData.Year == 2010
                                 select crimeData;

                    CrimeData vCrimeData = vCrime.First();
                    double vCrimePerCapita = (double)vCrimeData.ViolentCrimes / (double)vCrimeData.Population;
                    sb.Append($"Violent crime per capita rate (2010): {vCrimePerCapita}");
                    sb.Append(Environment.NewLine);

                    // Question Number 6
                    double avgMurders = CrimeDataList.Sum(x => x.Murders) / CrimeDataList.Count;
                    sb.Append($"Average murder per year (all years): {avgMurders}");
                    sb.Append(Environment.NewLine);

                    // Question Number 7
                    int murders1 = CrimeDataList
                    .Where(x => x.Year >= 1994 && x.Year <= 1997)
                    .Sum(y => y.Murders);
                    double avgMurders1 = murders1 / CrimeDataList.Count;
                    sb.Append($"Average murder per year (1994-1997): {avgMurders1}");
                    sb.Append(Environment.NewLine);

                    // Question Number 8
                    int murders2 = CrimeDataList
                    .Where(x => x.Year >= 2010 && x.Year <= 2014)
                    .Sum(y => y.Murders);
                    double avgMurders2 = murders2 / CrimeDataList.Count;
                    sb.Append($"Average murder per year (2010-2014): {avgMurders2}");
                    sb.Append(Environment.NewLine);

                    // Question Number 9
                    int minTheft = CrimeDataList
                    .Where(x => x.Year >= 1999 && x.Year <= 2004)
                    .Min(x => x.Thefts);
                    sb.Append($"Minimum thefts per year (1999-2004): {minTheft}");
                    sb.Append(Environment.NewLine);

                    // Question Number 10
                    int maxTheft = CrimeDataList
                    .Where(x => x.Year >= 1999 && x.Year <= 2004)
                    .Max(x => x.Thefts);
                    sb.Append($"Maximum thefts per year (1999-2004): {maxTheft}");
                    sb.Append(Environment.NewLine);

                    // Question Number 11
                    int yMaxVehicleTheft = CrimeDataList.OrderByDescending(x => x.MotorVehicleThefts).First().Year;
                    sb.Append($"Year of highest number of motor vehicle thefts: {yMaxVehicleTheft}");
                    sb.Append(Environment.NewLine);

                    using (var stream = new StreamWriter(filePath))
                    {
                        stream.Write(sb.ToString());
                    }
                    Console.WriteLine($"Written report successfully.");
                }
                else
                {
                    Console.WriteLine($"No data to write.");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error in writing report file.");
                throw ex;
            }
        }
    }
}

/* CrimeData.cs */

namespace ConsoleApp
{
    public class CrimeData
    {
        public int Year { get; set; }
        public int Population { get; set; }
        public int Murders { get; set; }
        public int Rapes { get; set; }
        public int Robberies { get; set; }
        public int ViolentCrimes { get; set; }
        public int Thefts { get; set; }
        public int MotorVehicleThefts { get; set; }
    }
}

/* Crime Data
Year,   Population,    Violent Crime,   Murder, Rape,   Robbery,    AA,         Property Crime, Burglary,   Theft,      Motor Vehicle Theft
1994,   260327021,     1857670,         23326,  102216, 618949,     1113179,    12131873,       2712774,    7879812,    1539287
1995,   262803276,     1798792,         21606,  97470,  580509,     1099207,    12063935,       2593784,    7997710,    1472441
1996,   265228572,     1688540,         19645,  96252,  535594      1037049,    11805323,       2506400,    7904685,    1394238
1997,   267783607,     1636096,         18208,  96153,  498534,     1023201,    11558475,       2460526,    7743760,    1354189
1998,   270248003,     1533887,         16974,  93144,  447186,     976583,     10951827,       2332735,    7376311,    1242781
1999,   272690813,     1426044,         15522,  89411,  409371,     911740,     10208334,       2100739,    6955520,    1152075
2000,   281421906,     1425486,         15586,  90178,  408016,     911706,     10182584,       2050992,    6971590,    1160002
2001,   285317559,     1439480,         16037,  90863,  423557,     909023,     10437189,       2116531,    7092267,    1228391
2002,   287973924,     1423677,         16229,  95235,  420806,     891407,     10455277,       2151252,    7057379,    1246646
2003,   290788976,     1383676,         16528,  93883,  414235,     859030,     10442862,       2154834,    7026802,    1261226
2004,   293656842,     1360088,         16148,  95089,  401470,     847381,     10319386,       2144446,    6937089,    1237851
2005,   296507061,     1390745,         16740,  94347,  417438,     862220,     10174754,       2155448,    6783447,    1235859
2006,   299398484,     1435123,         17309,  94472,  449246,     874096,     10019601,       2194993,    6626363,    1198245
2007,   301621157,     1422970,         17128,  92160,  447324,     866358,     9882212,        2190198,    6591542,    1100472
2008,   304059724,     1394461,         16465,  90750,  443563,     843683,     9774152,        2228887,    6586206,    959059
2009,   307006550,     1325896,         15399,  89241,  408742,     812514,     9337060,        2203313,    6338095,    795652
2010,   309330219,     1251248,         14722,  85593,  369089,     781844,     9112625,        2168459,    6204601,    739565
2011,   311587816,     1206005,         14661,  84175,  354746,     752423,     9052743,        2185140,    6151095,    716508
2012,   313873685,     1217057,         14856,  85141,  355051,     762009,     9001992,        2109932,    6168874,    723186
2013,   316128839,     1163146,         14196,  79770,  345031,     724149,     8632512,        1928465,    6004453,    699594
*/
