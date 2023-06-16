# CIS 556
United States of America Election Landscape Analysis (1976-2020)

**Introduction**

The union of the 50 sovereign states that represent the nation each exemplify the voices of their citizens, especially in a high-stakes federal election. The United States of America conducts a quadrennial presidential election to determine its president through a process called the Electoral College. This process does not necessarily represent the popular vote, as witnessed in the 2016 federal election. 

An investigation into the election day outcomes reveals much about its population its respective ideology. Throughout time, there is a shift of ideologies and preference on how the citizens want their government to conduct themselves.This inquisition is further analysed through the database curated by every election result from 1976 till 2020. Examining the state level significance in overall federal outcome is brought forward as swing state results like the 2000 Florida and 2020 Georgia results held great importance.

**Domain**

The dataset utilized in this analysis is from Massachusetts Institute of Technology. The Election Data and Science Lab (MEDSL) collected federal and state level results from the previous twelve election and a county centered analysisfor the elections in the 21st century. The county level dataset will be used to examine the historic close calls of the 2000 and 2020 elections.

U.S. President Election Results 1976-2020 [1]: 
 https://dataverse.harvard.edu/file.xhtml?fileId=4299753&version=6.0
Attributes in focus: year, state, candidate, party_detailed, candidatevotes and totalvotes.
U.S. County Presidential election Returns 2000-2020 [2]: 
 https://dataverse.harvard.edu/file.xhtml?fileId=4819117&version=9.0
Attributes in focus: year, state, county_name, candidate, party, candidatevotes and totalvotes. Candidate Information: Curated by me via Wikipedia.com Attributes in focus: year, candidate, candidate_home_state, running_mate, running_mate_home_state etc

**Cleaning**

Because we do not have more information about the candidate with entry as “OTHER”, we have decided to alter the entry to NULL. This data entry holds little significance to this investigation. Update president_election set candidate = NULL where candidate = 'OTHER'
There exist cases where the candidate’s name is not in the form of “Lastname, Firstname, Middle initial”. To keep consistency, we are sticking with Republican and Democratic candidate names from the candidate_information.csv. Altering the table with a join allows non major party candidates such as “INDEPENDENT” or “CONSERVATIVE” as well as entries with incorrect nomenclature such as “MITT, ROMNEY” instead of “ROMNEY, MITT” to be removed. There are even cases where votes went to a candidate, but the wrong party was entered. These cases were ignored as they are not valid votes.

CREATE TABLE DemorcraticVSRepublican AS 
(SELECT p.year as "Year", 
p.state as "State", 
p.state_po as "State ab.",
p.party_detailed as "Party",
p.candidate as "Candidate", 
i.home_state as "Candidate Home State", 
i.running_mate as "Running Mate", 
i.rm_home_state as "RM Home State", 
p.candidatevotes as "Votes", 
p.totalvotes as "Total Votes" from President_election_results p 
JOIN Candidate_information i ON p.candidate=i.candidate and 
p.year=i.year and p.writein = 'FALSE' ORDER by p.year);
Attached files

**DDL and DML statements**

(See attached file DMLStatements.txt and DDLStatements.txt)

**Queries**

**1. Names of former running mates of presidential candidates that ran for
presidency.**

SELECT Distinct n."Candidate" FROM DemorcraticVSRepublican n
JOIN DemorcraticVSRepublican m on m."Running Mate" = 
n."Candidate" WHERE n."Candidate" = m."Running Mate";

**2. Largest state victory. Largest county victory**

 **a. Federal level**

SELECT "Year", "Candidate", "Party", 
concat(Round(Cast(sum("Votes") as decimal)/sum("Total 
Votes")*100,3),'%') as "Vote%" from DemorcraticVSRepublican
WHERE "Party" IN ('DEMOCRAT','REPUBLICAN') GROUP BY "Candidate", 
"Year", "Party"
ORDER BY "Vote%" desc LIMIT 1;

 **b. State level**

SELECT "Year", "Candidate", "Party", "State",
Round(Cast(sum("Votes") as decimal)/sum("Total Votes")*100,3) as 
"Vote%" from DemorcraticVSRepublican
WHERE "Party" IN ('DEMOCRAT','REPUBLICAN') GROUP BY "Candidate", 
"Year", "Party","State"
ORDER BY "Vote%" desc LIMIT 1;

**3. What is the political party’s election performance every year?**

SELECT "Year", "Candidate", "Party", 
concat(Round(Cast(sum("Votes") as decimal)/sum("Total 
Votes")*100,3),'%') as "Vote%" from DemorcraticVSRepublican
WHERE "Party" IN ('DEMOCRAT','REPUBLICAN') GROUP BY "Candidate", 
"Year", "Party"
ORDER BY "Year", "Vote%" desc;

**4. Top 10 closest state election results. Alternatively: Which states had a less 
than 1% difference between top two candidates? (can be done by checking if
value is less than 0.01)**

SELECT dem."Year",dem."State", 
Concat(ROUND((CAST(abs(dem."Votes"-rep."Votes") as 
decimal)*100)/dem."Total Votes",4), '%') as Margin,
case when dem."Votes">rep."Votes"
 then 'DEMOCRAT'
 else 'REPUBLICAN'
end as Winner
FROM DemorcraticVSRepublican1 dem
LEFT join DemorcraticVSRepublican1 rep
on rep."State" = dem."State" and rep."Year" = dem."Year" 
where dem."Party" = 'DEMOCRAT' and rep."Party" = 'REPUBLICAN' 
ORDER BY Margin asc LIMIT 10;

**5. Which president or vice president lost their own state in federal election?**

**-- homestate loss dem**

SELECT dem."Year", dem."State", dem."Party", dem."Candidate", 
dem."Candidate Home State", dem."Running Mate", dem."RM Home 
State"
FROM DemorcraticVSRepublican dem
LEFT join DemorcraticVSRepublican rep
on rep."State" = dem."State" and rep."Year" = dem."Year"
where dem."Party" = 'DEMOCRAT' and rep."Party" = 'REPUBLICAN' 
and dem."Candidate Home State" = dem."State" and dem."Votes" < 
rep."Votes"
ORDER BY dem."Year";

**-- homestate loss rep**

SELECT rep."Year", rep."State", rep."Party", rep."Candidate", 
rep."Candidate Home State"
FROM DemorcraticVSRepublican rep
LEFT join DemorcraticVSRepublican dem
on dem."State" = rep."State" and dem."Year" = rep."Year"
where dem."Party" = 'DEMOCRAT' and rep."Party" = 'REPUBLICAN' 
and rep."Candidate Home State" = rep."State" and rep."Votes" < 
dem."Votes"
ORDER BY rep."Year";

**-- RM homestate loss dem**

Cast(sum("Votes") as decimal)
SELECT dem."Year", dem."State", dem."Party", dem."Running Mate", 
dem."RM Home State"
FROM DemorcraticVSRepublican dem
LEFT join DemorcraticVSRepublican rep
on rep."State" = dem."State" and rep."Year" = dem."Year"
where dem."Party" = 'DEMOCRAT' and rep."Party" = 'REPUBLICAN' 
and dem."RM Home State" = dem."State" and dem."Votes" < 
rep."Votes"
ORDER BY dem."Year";

**-- RM homestate loss rep**

SELECT rep."Year", rep."State", rep."Party", rep."Running Mate", 
rep."RM Home State"
FROM DemorcraticVSRepublican rep
LEFT join DemorcraticVSRepublican dem
on dem."State" = rep."State" and dem."Year" = rep."Year"
where dem."Party" = 'DEMOCRAT' and rep."Party" = 'REPUBLICAN' 
and rep."RM Home State" = rep."State" and rep."Votes" < 
dem."Votes"
ORDER BY rep."Year";

**6. Which states historically preferred a political party?
This code sets up a new table that presents winner at state level.**

SELECT f."State" , f."Year",
case when f."Votes"<s."Votes"
 then 'DEMOCRAT'
 else 'REPUBLICAN'
end as "winner" into StateWinners
from DemorcraticVSRepublican f
Inner Join DemorcraticVSRepublican s on s."Year"=f."Year" and 
f."State"=s."State" and f."Party"='REPUBLICAN' and 
s."Party"='DEMOCRAT' Order by f."State";

**a. Red states vs. Blue states: 100% preference**

SELECT "State", "winner" from StateWinners 
WHERE "winner" in ('DEMOCRAT', 'REPUBLICAN') 
Group by "State", "winner" HAVING Count("winner")/12 = 1 ORDER 
by "State";

**b. All but once**

SELECT "State", "winner" from StateWinners 
WHERE "winner" in ('DEMOCRAT', 'REPUBLICAN') 
Group by "State", "winner" HAVING Count("winner") in (11) ORDER 
by "State";

**c. Purple States: Half the time**

SELECT "State" from StateWinners 
WHERE "winner" in ('DEMOCRAT') 
Group by "State", "winner" HAVING Count("winner") in (6) ORDER 
by "State"

**Analysis**

Presidential election should represent its citizen’s voice. With this database holding fundamental data, much information can be extracted such as Ronald Reagan hosting the greatest victory at 58.46% of the nations votes as per query 2. Query 3 details the popular vote in the federal election, rather than the electoral college system which is practices under the constitution. The data base collects the raw data from election ballots, which in certain cases, is not the primary marker of the elected presidential. The state level results determine if candidate receives electoral college points which is later accumulated to elect the president. As many states highly favoured one side over the other, query 6 showcases the disparity in states performance. The idea of strongholds for certain political parties is a significant factor in structuring an influential campaign. Nine states have always voted republican since 1976 while only one voted democratic. We can also change the threshold and see which states highly favour one party of the other as shown in Figure 2b. This insight can be extract with a simple SQL query statement where more complex statements can discover. Throughout the recent history, the American political landscape has been dominated with the two major political parties: Republican Party and Democratic Party. 

Query 4 invokes further investigation amongst these two parties close battle focusing on cases with less than 1% differential. According to Election Management Guidelines states may request or be automatically required to do a recount if margin of victory falls under certain threshold. [5] Most famous instance was the 2000 Florida results with a differential of 0.009% which was later pushed to Supreme Court as this one state’s decision ultimately determined the president. Bush vs. Gore famously let the fate of the nation be determined by the government rather than the people.[6] Query 4 showcase many similar cases with the most recent being the 2020 Georgia election. 

**Case Study: 2000 Florida**

This close call was historic in its influence on the 2000 federal election as it had a major recount dispute. With George W, Bush initially winning the State with 537 votes over Al Gore, steps were implemented for a recount in the supreme court. [6] As this remaining state with 25 electoral votes determined the president-elect, a careful vote counting was conducted. The importance data collection process and storage are evident. Further investigation into the county level of this state showed that the close battle was also affected by Ralph Nader of the Green party managed to collect 1.6% of the states total votes that could have gone toward any of the two primary political parties.

**Case Study: 2020 Georgia**

With the 2020 election held in the midst of the COVID-19 pandemic, numerous voting methods were put into effect to accommodate everyone’s vote. The vote tallying process invoked many concerns and fear of fraudulent behaviour. A special consideration must be made to the mail-in ballots on top of the advanced polls and election day ballots.This timeline prompted parties to exercise their rights in protecting a win. In the case of Georgia, the republican party held a 28.39% to 25.02% lead over the Democratic party in advanced voting. On election day, Republican picked up an 11.76% increase while the Democrats grabbed a 7.3%. This aggregate heavily favored the Republicans. But the interesting mode of voting that skew towards Democrats is provisional ballots. This voting exception allows voters to vote without sufficient requirement at the time under the circumstances are met at a set date. According to the Georgia voters information guide, provisional ballots can be counted if problem is solved within three days after election day.[4] These votes are subject to being held out until verification is presented. With this threatening the Republican lead, many attempts to halt the count was implemented. With mail-in ballots, also known as absentee ballots, being counted post election day, it is evident that people who mailed in their votes rather than waiting at the polls during a pandemic, highly favoured the Democratic candidate of Joe Biden 17% to Donald Trumps 9%. The impending results showcased that back and forth of election day ballot counting and how modes can alter and affect the results. The demographic behavior is shown beyond the political party as mode and approach to 2020.

**Conclusion**

This in-depth database holds great information regarding the presidential election results. With the power or data manipulation and relational algebra, and the functions and tools taught by Professor Niccolo Meneghetti helped me further my curiosity in the political landscape of USA. The utilization of such data provides explanations of trends and habits that can be applied in campaign strategy, demographic analysis, and voter security.
