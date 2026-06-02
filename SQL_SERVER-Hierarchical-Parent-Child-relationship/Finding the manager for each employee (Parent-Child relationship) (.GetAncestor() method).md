Exercise #: Finding the manager for each employee based on Parent-Child relationship


Solution: We use the built-in method .GetAncestor() to navigate the organization's hierarchical structure for all company's levels, and retrieve the hierarchyid data type used to find a parent or higher-level ancestor node in a tree structure.


Tables used:
1) [AdventureWorks2022].[Person].[Person]
2) [AdventureWorks2022].[HumanResources].[Employee]



USE AdventureWorks2022;
GO

WITH Management AS											-- IncludingUpperManagement
(
SELECT	EmployeePerson.[BusinessEntityID] AS BusinessEntityID																-- Employee Details
	  , CONCAT(EmployeePerson.[FirstName], ' ', EmployeePerson.[MiddleName], ' ', EmployeePerson.[LastName]) AS EmployeeName
	  ,EmployeeTitle.[JobTitle] AS EmployeeTitle																		    -- Managers Details
	  ,EmployeeTitle.[OrganizationNode].GetAncestor(0) AS OwnNode
	  ,EmployeeTitle.[OrganizationNode].GetAncestor(1) AS ManagerNode
	  ,CAST(NULL AS NVARCHAR(150)) AS ManagerName
	  ,CAST(NULL AS NVARCHAR(50)) AS ManagerTitle
FROM [AdventureWorks2022].[Person].[Person] AS EmployeePerson
RIGHT JOIN [AdventureWorks2022].[HumanResources].[Employee] AS EmployeeTitle
ON EmployeePerson.[BusinessEntityID] = EmployeeTitle.[BusinessEntityID]
WHERE EmployeeTitle.[OrganizationNode].GetAncestor(1) = 0x OR EmployeeTitle.[OrganizationNode] IS NULL
),
Management2 AS												-- IncludingUpperManagementPopulatingNULLs
(
SELECT	Management.BusinessEntityID
	  ,Management.EmployeeName
	  ,Management.EmployeeTitle
	  ,CASE WHEN OwnNode IS NOT NULL THEN OwnNode ELSE CAST(NULL AS NVARCHAR(150)) END AS OwnNode
	  ,CASE WHEN ManagerNode IS NOT NULL THEN ManagerNode ELSE CAST(NULL AS NVARCHAR(150)) END AS ManagerNode
	  ,CASE WHEN ManagerNode = 0x THEN 'Ken J Sánchez' ELSE CAST(NULL AS NVARCHAR(150)) END AS ManagerName
	  ,CASE WHEN ManagerNode IS NULL THEN 'N/A' ELSE '' END AS ManagerTitle
FROM Management
),
Employees AS												-- IncludingRestEmployeesExcludingUpperManagement
(
SELECT	EmployeePerson.[BusinessEntityID] AS BusinessEntityID						-- Employee Details
	  ,CONCAT(EmployeePerson.[FirstName], ' ', EmployeePerson.[MiddleName], ' ', EmployeePerson.[LastName]) AS EmployeeName
	  ,EmployeeTitle.[JobTitle] AS EmployeeTitle							-- Manager Details
	  ,EmployeeTitle.[OrganizationNode].GetAncestor(0) AS OwnNode
	  ,ManagerTitle.[OrganizationNode].GetAncestor(1) AS ManagerNode
	  ,CONCAT(ManagerPerson.[FirstName], ' ', ManagerPerson.[MiddleName], ' ', ManagerPerson.[LastName]) AS ManagerName
	  ,ManagerTitle.[JobTitle] AS ManagerTitle
FROM [AdventureWorks2022].[Person].[Person] AS EmployeePerson
RIGHT JOIN [AdventureWorks2022].[HumanResources].[Employee] AS EmployeeTitle
ON EmployeePerson.[BusinessEntityID] = EmployeeTitle.[BusinessEntityID]
INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS ManagerTitle
ON EmployeeTitle.[OrganizationNode].GetAncestor(1) = ManagerTitle.[OrganizationNode]
LEFT JOIN [AdventureWorks2022].[Person].[Person] AS ManagerPerson
ON ManagerTitle.[BusinessEntityID] = ManagerPerson.[BusinessEntityID]
)
SELECT	BusinessEntityID
	  ,EmployeeName
	  ,EmployeeTitle
	  ,OwnNode
	  ,ManagerNode
	  ,ManagerName
	  ,ManagerTitle
FROM Management2
UNION ALL 
SELECT	BusinessEntityID
	  ,EmployeeName
	  ,EmployeeTitle
	  ,OwnNode
	  ,ManagerNode
	  ,ManagerName
	  ,ManagerTitle
FROM Employees;


  -- OUTPUT
BusinessEntityID	EmployeeName	EmployeeTitle	OwnNode	ManagerNode	ManagerName	ManagerTitle
1	Ken J Sánchez	Chief Executive Officer	NULL	NULL	NULL	N/A
2	Terri Lee Duffy	Vice President of Engineering	0x58	0x	Ken J Sánchez	
16	David M Bradley	Marketing Manager	0x68	0x	Ken J Sánchez	
25	James R Hamilton	Vice President of Production	0x78	0x	Ken J Sánchez	
234	Laura F Norman	Chief Financial Officer	0x84	0x	Ken J Sánchez	
263	Jean E Trenary	Information Services Manager	0x8C	0x	Ken J Sánchez	
273	Brian S Welcker	Vice President of Sales	0x94	0x	Ken J Sánchez	
3	Roberto  Tamburello	Engineering Manager	0x5AC0	0x	Terri Lee Duffy	Vice President of Engineering
4	Rob  Walters	Senior Tool Designer	0x5AD6	0x58	Roberto  Tamburello	Engineering Manager
5	Gail A Erickson	Design Engineer	0x5ADA	0x58	Roberto  Tamburello	Engineering Manager
6	Jossef H Goldberg	Design Engineer	0x5ADE	0x58	Roberto  Tamburello	Engineering Manager
7	Dylan A Miller	Research and Development Manager	0x5AE1	0x58	Roberto  Tamburello	Engineering Manager
8	Diane L Margheim	Research and Development Engineer	0x5AE158	0x5AC0	Dylan A Miller	Research and Development Manager
9	Gigi N Matthew	Research and Development Engineer	0x5AE168	0x5AC0	Dylan A Miller	Research and Development Manager
10	Michael  Raheem	Research and Development Manager	0x5AE178	0x5AC0	Dylan A Miller	Research and Development Manager
11	Ovidiu V Cracium	Senior Tool Designer	0x5AE3	0x58	Roberto  Tamburello	Engineering Manager
12	Thierry B D'Hers	Tool Designer	0x5AE358	0x5AC0	Ovidiu V Cracium	Senior Tool Designer
13	Janice M Galvin	Tool Designer	0x5AE368	0x5AC0	Ovidiu V Cracium	Senior Tool Designer
14	Michael I Sullivan	Senior Design Engineer	0x5AE5	0x58	Roberto  Tamburello	Engineering Manager
15	Sharon B Salavaria	Design Engineer	0x5AE7	0x58	Roberto  Tamburello	Engineering Manager
17	Kevin F Brown	Marketing Assistant	0x6AC0	0x	David M Bradley	Marketing Manager
18	John L Wood	Marketing Specialist	0x6B40	0x	David M Bradley	Marketing Manager
19	Mary A Dempsey	Marketing Assistant	0x6BC0	0x	David M Bradley	Marketing Manager
20	Wanida M Benshoof	Marketing Assistant	0x6C20	0x	David M Bradley	Marketing Manager
21	Terry J Eminhizer	Marketing Specialist	0x6C60	0x	David M Bradley	Marketing Manager
22	Sariya E Harnpadoungsataya	Marketing Specialist	0x6CA0	0x	David M Bradley	Marketing Manager
23	Mary E Gibson	Marketing Specialist	0x6CE0	0x	David M Bradley	Marketing Manager
24	Jill A Williams	Marketing Specialist	0x6D10	0x	David M Bradley	Marketing Manager
26	Peter J Krebs	Production Control Manager	0x7AC0	0x	James R Hamilton	Vice President of Production
27	Jo A Brown	Production Supervisor - WC60	0x7AD6	0x78	Peter J Krebs	Production Control Manager
28	Guy R Gilbert	Production Technician - WC60	0x7AD6B0	0x7AC0	Jo A Brown	Production Supervisor - WC60
29	Mark K McArthur	Production Technician - WC60	0x7AD6D0	0x7AC0	Jo A Brown	Production Supervisor - WC60
30	Britta L Simon	Production Technician - WC60	0x7AD6F0	0x7AC0	Jo A Brown	Production Supervisor - WC60
31	Margie W Shoop	Production Technician - WC60	0x7AD708	0x7AC0	Jo A Brown	Production Supervisor - WC60
32	Rebecca A Laszlo	Production Technician - WC60	0x7AD718	0x7AC0	Jo A Brown	Production Supervisor - WC60
33	Annik O Stahl	Production Technician - WC60	0x7AD728	0x7AC0	Jo A Brown	Production Supervisor - WC60
34	Suchitra O Mohan	Production Technician - WC60	0x7AD738	0x7AC0	Jo A Brown	Production Supervisor - WC60
35	Brandon G Heidepriem	Production Technician - WC60	0x7AD744	0x7AC0	Jo A Brown	Production Supervisor - WC60
36	Jose R Lugo	Production Technician - WC60	0x7AD74C	0x7AC0	Jo A Brown	Production Supervisor - WC60
37	Chris O Okelberry	Production Technician - WC60	0x7AD754	0x7AC0	Jo A Brown	Production Supervisor - WC60
38	Kim B Abercrombie	Production Technician - WC60	0x7AD75C	0x7AC0	Jo A Brown	Production Supervisor - WC60
39	Ed R Dudenhoefer	Production Technician - WC60	0x7AD764	0x7AC0	Jo A Brown	Production Supervisor - WC60
40	JoLynn M Dobney	Production Supervisor - WC60	0x7ADA	0x78	Peter J Krebs	Production Control Manager
41	Bryan  Baker	Production Technician - WC60	0x7ADAB0	0x7AC0	JoLynn M Dobney	Production Supervisor - WC60
42	James D Kramer	Production Technician - WC60	0x7ADAD0	0x7AC0	JoLynn M Dobney	Production Supervisor - WC60
43	Nancy A Anderson	Production Technician - WC60	0x7ADAF0	0x7AC0	JoLynn M Dobney	Production Supervisor - WC60
44	Simon D Rapier	Production Technician - WC60	0x7ADB08	0x7AC0	JoLynn M Dobney	Production Supervisor - WC60
45	Thomas R Michaels	Production Technician - WC60	0x7ADB18	0x7AC0	JoLynn M Dobney	Production Supervisor - WC60
46	Eugene O Kogan	Production Technician - WC60	0x7ADB28	0x7AC0	JoLynn M Dobney	Production Supervisor - WC60
47	Andrew R Hill	Production Supervisor - WC10	0x7ADE	0x78	Peter J Krebs	Production Control Manager
48	Ruth Ann Ellerbrock	Production Technician - WC10	0x7ADEB0	0x7AC0	Andrew R Hill	Production Supervisor - WC10
49	Barry K Johnson	Production Technician - WC10	0x7ADED0	0x7AC0	Andrew R Hill	Production Supervisor - WC10
50	Sidney M Higa	Production Technician - WC10	0x7ADEF0	0x7AC0	Andrew R Hill	Production Supervisor - WC10
51	Jeffrey L Ford	Production Technician - WC10	0x7ADF08	0x7AC0	Andrew R Hill	Production Supervisor - WC10
52	Doris M Hartwig	Production Technician - WC10	0x7ADF18	0x7AC0	Andrew R Hill	Production Supervisor - WC10
53	Diane R Glimp	Production Technician - WC10	0x7ADF28	0x7AC0	Andrew R Hill	Production Supervisor - WC10
54	Bonnie N Kearney	Production Technician - WC10	0x7ADF38	0x7AC0	Andrew R Hill	Production Supervisor - WC10
55	Taylor R Maxwell	Production Supervisor - WC50	0x7AE1	0x78	Peter J Krebs	Production Control Manager
56	Denise H Smith	Production Technician - WC50	0x7AE158	0x7AC0	Taylor R Maxwell	Production Supervisor - WC50
57	Frank T Miller	Production Technician - WC50	0x7AE168	0x7AC0	Taylor R Maxwell	Production Supervisor - WC50
58	Kendall C Keil	Production Technician - WC50	0x7AE178	0x7AC0	Taylor R Maxwell	Production Supervisor - WC50
59	Bob N Hohman	Production Technician - WC50	0x7AE184	0x7AC0	Taylor R Maxwell	Production Supervisor - WC50
60	Pete C Male	Production Technician - WC50	0x7AE18C	0x7AC0	Taylor R Maxwell	Production Supervisor - WC50
61	Diane H Tibbott	Production Technician - WC50	0x7AE194	0x7AC0	Taylor R Maxwell	Production Supervisor - WC50
62	John T Campbell	Production Supervisor - WC60	0x7AE3	0x78	Peter J Krebs	Production Control Manager
63	Maciej W Dusza	Production Technician - WC60	0x7AE358	0x7AC0	John T Campbell	Production Supervisor - WC60
64	Michael J Zwilling	Production Technician - WC60	0x7AE368	0x7AC0	John T Campbell	Production Supervisor - WC60
65	Randy T Reeves	Production Technician - WC60	0x7AE378	0x7AC0	John T Campbell	Production Supervisor - WC60
66	Karan R Khanna	Production Technician - WC60	0x7AE384	0x7AC0	John T Campbell	Production Supervisor - WC60
67	Jay G Adams	Production Technician - WC60	0x7AE38C	0x7AC0	John T Campbell	Production Supervisor - WC60
68	Charles B Fitzgerald	Production Technician - WC60	0x7AE394	0x7AC0	John T Campbell	Production Supervisor - WC60
69	Steve F Masters	Production Technician - WC60	0x7AE39C	0x7AC0	John T Campbell	Production Supervisor - WC60
70	David J Ortiz	Production Technician - WC60	0x7AE3A2	0x7AC0	John T Campbell	Production Supervisor - WC60
71	Michael Sean Ray	Production Supervisor - WC30	0x7AE5	0x78	Peter J Krebs	Production Control Manager
72	Steven T Selikoff	Production Technician - WC30	0x7AE558	0x7AC0	Michael Sean Ray	Production Supervisor - WC30
73	Carole M Poland	Production Technician - WC30	0x7AE568	0x7AC0	Michael Sean Ray	Production Supervisor - WC30
74	Bjorn M Rettig	Production Technician - WC30	0x7AE578	0x7AC0	Michael Sean Ray	Production Supervisor - WC30
75	Michiko F Osada	Production Technician - WC30	0x7AE584	0x7AC0	Michael Sean Ray	Production Supervisor - WC30
76	Carol M Philips	Production Technician - WC30	0x7AE58C	0x7AC0	Michael Sean Ray	Production Supervisor - WC30
77	Merav A Netz	Production Technician - WC30	0x7AE594	0x7AC0	Michael Sean Ray	Production Supervisor - WC30
78	Reuben H D'sa	Production Supervisor - WC40	0x7AE7	0x78	Peter J Krebs	Production Control Manager
79	Eric L Brown	Production Technician - WC40	0x7AE758	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
80	Sandeep P Kaliyath	Production Technician - WC40	0x7AE768	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
81	Mihail U Frintu	Production Technician - WC40	0x7AE778	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
82	Jack T Creasey	Production Technician - WC40	0x7AE784	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
83	Patrick M Cook	Production Technician - WC40	0x7AE78C	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
84	Frank R Martinez	Production Technician - WC40	0x7AE794	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
85	Brian Richard Goldstein	Production Technician - WC40	0x7AE79C	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
86	Ryan L Cornelsen	Production Technician - WC40	0x7AE7A2	0x7AC0	Reuben H D'sa	Production Supervisor - WC40
87	Cristian K Petculescu	Production Supervisor - WC10	0x7AE880	0x78	Peter J Krebs	Production Control Manager
88	Betsy A Stadick	Production Technician - WC10	0x7AE8AC	0x7AC0	Cristian K Petculescu	Production Supervisor - WC10
89	Patrick C Wedge	Production Technician - WC10	0x7AE8B4	0x7AC0	Cristian K Petculescu	Production Supervisor - WC10
90	Danielle C Tiedt	Production Technician - WC10	0x7AE8BC	0x7AC0	Cristian K Petculescu	Production Supervisor - WC10
91	Kimberly B Zimmerman	Production Technician - WC10	0x7AE8C2	0x7AC0	Cristian K Petculescu	Production Supervisor - WC10
92	Tom M Vande Velde	Production Technician - WC10	0x7AE8C6	0x7AC0	Cristian K Petculescu	Production Supervisor - WC10
93	Kok-Ho T Loh	Production Supervisor - WC50	0x7AE980	0x78	Peter J Krebs	Production Control Manager
94	Russell  Hunter	Production Technician - WC50	0x7AE9AC	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
95	Jim H Scardelis	Production Technician - WC50	0x7AE9B4	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
96	Elizabeth I Keyser	Production Technician - WC50	0x7AE9BC	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
97	Mandar H Samant	Production Technician - WC50	0x7AE9C2	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
98	Sameer A Tejani	Production Technician - WC50	0x7AE9C6	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
99	Nuan  Yu	Production Technician - WC50	0x7AE9CA	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
100	Lolan B Song	Production Technician - WC50	0x7AE9CE	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
101	Houman N Pournasseh	Production Technician - WC50	0x7AE9D1	0x7AC0	Kok-Ho T Loh	Production Supervisor - WC50
102	Zheng W Mu	Production Supervisor - WC10	0x7AEA80	0x78	Peter J Krebs	Production Control Manager
103	Ebru N Ersan	Production Technician - WC10	0x7AEAAC	0x7AC0	Zheng W Mu	Production Supervisor - WC10
104	Mary R Baker	Production Technician - WC10	0x7AEAB4	0x7AC0	Zheng W Mu	Production Supervisor - WC10
105	Kevin M Homer	Production Technician - WC10	0x7AEABC	0x7AC0	Zheng W Mu	Production Supervisor - WC10
106	John T Kane	Production Technician - WC10	0x7AEAC2	0x7AC0	Zheng W Mu	Production Supervisor - WC10
107	Christopher E Hill	Production Technician - WC10	0x7AEAC6	0x7AC0	Zheng W Mu	Production Supervisor - WC10
108	Jinghao K Liu	Production Supervisor - WC50	0x7AEB80	0x78	Peter J Krebs	Production Control Manager
109	Alice O Ciccu	Production Technician - WC50	0x7AEBAC	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
110	Jun T Cao	Production Technician - WC50	0x7AEBB4	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
111	Suroor R Fatima	Production Technician - WC50	0x7AEBBC	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
112	John P Evans	Production Technician - WC50	0x7AEBC2	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
113	Linda K Moschell	Production Technician - WC50	0x7AEBC6	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
114	Mindaugas J Krapauskas	Production Technician - WC50	0x7AEBCA	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
115	Angela W Barbariol	Production Technician - WC50	0x7AEBCE	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
116	Michael W Patten	Production Technician - WC50	0x7AEBD1	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
117	Chad W Niswonger	Production Technician - WC50	0x7AEBD3	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
118	Don L Hall	Production Technician - WC50	0x7AEBD5	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
119	Michael T Entin	Production Technician - WC50	0x7AEBD7	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
120	Kitti H Lertpiriyasuwat	Production Technician - WC50	0x7AEBD9	0x7AC0	Jinghao K Liu	Production Supervisor - WC50
121	Pilar G Ackerman	Shipping and Receiving Supervisor	0x7AEC80	0x78	Peter J Krebs	Production Control Manager
122	Susan W Eaton	Stocker	0x7AECAC	0x7AC0	Pilar G Ackerman	Shipping and Receiving Supervisor
123	Vamsi N Kuppa	Shipping and Receiving Clerk	0x7AECB4	0x7AC0	Pilar G Ackerman	Shipping and Receiving Supervisor
124	Kim T Ralls	Stocker	0x7AECBC	0x7AC0	Pilar G Ackerman	Shipping and Receiving Supervisor
125	Matthias T Berndt	Shipping and Receiving Clerk	0x7AECC2	0x7AC0	Pilar G Ackerman	Shipping and Receiving Supervisor
126	Jimmy T Bischoff	Stocker	0x7AECC6	0x7AC0	Pilar G Ackerman	Shipping and Receiving Supervisor
127	David P Hamilton	Production Supervisor - WC40	0x7AED80	0x78	Peter J Krebs	Production Control Manager
128	Paul B Komosinski	Production Technician - WC40	0x7AEDAC	0x7AC0	David P Hamilton	Production Supervisor - WC40
129	Gary W Yukish	Production Technician - WC40	0x7AEDB4	0x7AC0	David P Hamilton	Production Supervisor - WC40
130	Rob T Caron	Production Technician - WC40	0x7AEDBC	0x7AC0	David P Hamilton	Production Supervisor - WC40
131	Baris F Cetinok	Production Technician - WC40	0x7AEDC2	0x7AC0	David P Hamilton	Production Supervisor - WC40
132	Nicole B Holliday	Production Technician - WC40	0x7AEDC6	0x7AC0	David P Hamilton	Production Supervisor - WC40
133	Michael L Rothkugel	Production Technician - WC40	0x7AEDCA	0x7AC0	David P Hamilton	Production Supervisor - WC40
134	Eric  Gubbels	Production Supervisor - WC20	0x7AEE80	0x78	Peter J Krebs	Production Control Manager
135	Ivo William Salmre	Production Technician - WC20	0x7AEEAC	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
136	Sylvester A Valdez	Production Technician - WC20	0x7AEEB4	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
137	Anibal T Sousa	Production Technician - WC20	0x7AEEBC	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
138	Samantha H Smith	Production Technician - WC20	0x7AEEC2	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
139	Hung-Fu T Ting	Production Technician - WC20	0x7AEEC6	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
140	Prasanna E Samarawickrama	Production Technician - WC20	0x7AEECA	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
141	Min G Su	Production Technician - WC20	0x7AEECE	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
142	Olinda C Turner	Production Technician - WC20	0x7AEED1	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
143	Krishna  Sunkammurali	Production Technician - WC20	0x7AEED3	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
144	Paul R Singh	Production Technician - WC20	0x7AEED5	0x7AC0	Eric  Gubbels	Production Supervisor - WC20
145	Cynthia S Randall	Production Supervisor - WC30	0x7AEF80	0x78	Peter J Krebs	Production Control Manager
146	Jian Shuo  Wang	Production Technician - WC30	0x7AEFAC	0x7AC0	Cynthia S Randall	Production Supervisor - WC30
147	Sandra  Reátegui Alayo	Production Technician - WC30	0x7AEFB4	0x7AC0	Cynthia S Randall	Production Supervisor - WC30
148	Jason M Watters	Production Technician - WC30	0x7AEFBC	0x7AC0	Cynthia S Randall	Production Supervisor - WC30
149	Andy M Ruth	Production Technician - WC30	0x7AEFC2	0x7AC0	Cynthia S Randall	Production Supervisor - WC30
150	Michael T Vanderhyde	Production Technician - WC30	0x7AEFC6	0x7AC0	Cynthia S Randall	Production Supervisor - WC30
151	Rostislav E Shabalin	Production Technician - WC30	0x7AEFCA	0x7AC0	Cynthia S Randall	Production Supervisor - WC30
152	Yuhong L Li	Production Supervisor - WC20	0x7AF044	0x78	Peter J Krebs	Production Control Manager
153	Hanying P Feng	Production Technician - WC20	0x7AF04560	0x7AC0	Yuhong L Li	Production Supervisor - WC20
154	Raymond K Sam	Production Technician - WC20	0x7AF045A0	0x7AC0	Yuhong L Li	Production Supervisor - WC20
155	Fadi K Fakhouri	Production Technician - WC20	0x7AF045E0	0x7AC0	Yuhong L Li	Production Supervisor - WC20
156	Lane M Sacksteder	Production Technician - WC20	0x7AF04610	0x7AC0	Yuhong L Li	Production Supervisor - WC20
157	Linda A Randall	Production Technician - WC20	0x7AF04630	0x7AC0	Yuhong L Li	Production Supervisor - WC20
158	Shelley N Dyck	Production Technician - WC20	0x7AF04650	0x7AC0	Yuhong L Li	Production Supervisor - WC20
159	Terrence W Earls	Production Technician - WC20	0x7AF04670	0x7AC0	Yuhong L Li	Production Supervisor - WC20
160	Jeff V Hay	Production Supervisor - WC45	0x7AF04C	0x78	Peter J Krebs	Production Control Manager
161	Kirk J Koenigsbauer	Production Technician - WC45	0x7AF04D60	0x7AC0	Jeff V Hay	Production Supervisor - WC45
162	Laura C Steele	Production Technician - WC45	0x7AF04DA0	0x7AC0	Jeff V Hay	Production Supervisor - WC45
163	Alex M Nayberg	Production Technician - WC45	0x7AF04DE0	0x7AC0	Jeff V Hay	Production Supervisor - WC45
164	Andrew M Cencini	Production Technician - WC45	0x7AF04E10	0x7AC0	Jeff V Hay	Production Supervisor - WC45
165	Chris T Preston	Production Technician - WC45	0x7AF04E30	0x7AC0	Jeff V Hay	Production Supervisor - WC45
166	Jack S Richins	Production Supervisor - WC30	0x7AF054	0x78	Peter J Krebs	Production Control Manager
167	David N Johnson	Production Technician - WC30	0x7AF05560	0x7AC0	Jack S Richins	Production Supervisor - WC30
168	Garrett R Young	Production Technician - WC30	0x7AF055A0	0x7AC0	Jack S Richins	Production Supervisor - WC30
169	Susan A Metters	Production Technician - WC30	0x7AF055E0	0x7AC0	Jack S Richins	Production Supervisor - WC30
170	George Z Li	Production Technician - WC30	0x7AF05610	0x7AC0	Jack S Richins	Production Supervisor - WC30
171	David A Yalovsky	Production Technician - WC30	0x7AF05630	0x7AC0	Jack S Richins	Production Supervisor - WC30
172	Marc J Ingle	Production Technician - WC30	0x7AF05650	0x7AC0	Jack S Richins	Production Supervisor - WC30
173	Eugene R Zabokritski	Production Technician - WC30	0x7AF05670	0x7AC0	Jack S Richins	Production Supervisor - WC30
174	Benjamin R Martin	Production Technician - WC30	0x7AF05688	0x7AC0	Jack S Richins	Production Supervisor - WC30
175	Reed T Koch	Production Technician - WC30	0x7AF05698	0x7AC0	Jack S Richins	Production Supervisor - WC30
176	David Oliver Lawrence	Production Technician - WC30	0x7AF056A8	0x7AC0	Jack S Richins	Production Supervisor - WC30
177	Russell M King	Production Technician - WC30	0x7AF056B8	0x7AC0	Jack S Richins	Production Supervisor - WC30
178	John N Frum	Production Technician - WC30	0x7AF056C8	0x7AC0	Jack S Richins	Production Supervisor - WC30
179	Jan S Miksovsky	Production Technician - WC30	0x7AF056D8	0x7AC0	Jack S Richins	Production Supervisor - WC30
180	Katie L McAskill-White	Production Supervisor - WC20	0x7AF05C	0x78	Peter J Krebs	Production Control Manager
181	Michael T Hines	Production Technician - WC20	0x7AF05D60	0x7AC0	Katie L McAskill-White	Production Supervisor - WC20
182	Nitin S Mirchandani	Production Technician - WC20	0x7AF05DA0	0x7AC0	Katie L McAskill-White	Production Supervisor - WC20
183	Barbara S Decker	Production Technician - WC20	0x7AF05DE0	0x7AC0	Katie L McAskill-White	Production Supervisor - WC20
184	John Y Chen	Production Technician - WC20	0x7AF05E10	0x7AC0	Katie L McAskill-White	Production Supervisor - WC20
185	Stefen A Hesse	Production Technician - WC20	0x7AF05E30	0x7AC0	Katie L McAskill-White	Production Supervisor - WC20
186	Shane S Kim	Production Supervisor - WC45	0x7AF064	0x78	Peter J Krebs	Production Control Manager
187	Yvonne S McKay	Production Technician - WC45	0x7AF06560	0x7AC0	Shane S Kim	Production Supervisor - WC45
188	Douglas B Hite	Production Technician - WC45	0x7AF065A0	0x7AC0	Shane S Kim	Production Supervisor - WC45
189	Janeth M Esteves	Production Technician - WC45	0x7AF065E0	0x7AC0	Shane S Kim	Production Supervisor - WC45
190	Robert J Rounthwaite	Production Technician - WC45	0x7AF06610	0x7AC0	Shane S Kim	Production Supervisor - WC45
191	Lionel C Penuchot	Production Technician - WC45	0x7AF06630	0x7AC0	Shane S Kim	Production Supervisor - WC45
192	Brenda M Diaz	Production Supervisor - WC40	0x7AF06C	0x78	Peter J Krebs	Production Control Manager
193	Alejandro E McGuel	Production Technician - WC40	0x7AF06D60	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
194	Fred T Northup	Production Technician - WC40	0x7AF06DA0	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
195	Kevin H Liu	Production Technician - WC40	0x7AF06DE0	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
196	Shammi G Mohamed	Production Technician - WC40	0x7AF06E10	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
197	Rajesh M Patel	Production Technician - WC40	0x7AF06E30	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
198	Lorraine O Nay	Production Technician - WC40	0x7AF06E50	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
199	Paula R Nartker	Production Technician - WC40	0x7AF06E70	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
200	Frank T Lee	Production Technician - WC40	0x7AF06E88	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
201	Brian T Lloyd	Production Technician - WC40	0x7AF06E98	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
202	Tawana G Nusbaum	Production Technician - WC40	0x7AF06EA8	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
203	Ken L Myer	Production Technician - WC40	0x7AF06EB8	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
204	Gabe B Mares	Production Technician - WC40	0x7AF06EC8	0x7AC0	Brenda M Diaz	Production Supervisor - WC40
205	Lori A Kane	Production Supervisor - WC45	0x7AF074	0x78	Peter J Krebs	Production Control Manager
206	Stuart V Munson	Production Technician - WC45	0x7AF07560	0x7AC0	Lori A Kane	Production Supervisor - WC45
207	Greg F Alderson	Production Technician - WC45	0x7AF075A0	0x7AC0	Lori A Kane	Production Supervisor - WC45
208	Scott R Gode	Production Technician - WC45	0x7AF075E0	0x7AC0	Lori A Kane	Production Supervisor - WC45
209	Kathie E Flood	Production Technician - WC45	0x7AF07610	0x7AC0	Lori A Kane	Production Supervisor - WC45
210	Belinda M Newman	Production Technician - WC45	0x7AF07630	0x7AC0	Lori A Kane	Production Supervisor - WC45
211	Hazem E Abolrous	Quality Assurance Manager	0x7B40	0x	James R Hamilton	Vice President of Production
212	Peng J Wu	Quality Assurance Supervisor	0x7B56	0x78	Hazem E Abolrous	Quality Assurance Manager
213	Sootha T Charncherngkha	Quality Assurance Technician	0x7B56B0	0x7B40	Peng J Wu	Quality Assurance Supervisor
214	Andreas T Berglund	Quality Assurance Technician	0x7B56D0	0x7B40	Peng J Wu	Quality Assurance Supervisor
215	Mark L Harrington	Quality Assurance Technician	0x7B56F0	0x7B40	Peng J Wu	Quality Assurance Supervisor
216	Sean P Alexander	Quality Assurance Technician	0x7B5708	0x7B40	Peng J Wu	Quality Assurance Supervisor
217	Zainal T Arifin	Document Control Manager	0x7B5A	0x78	Hazem E Abolrous	Quality Assurance Manager
218	Tengiz N Kharatishvili	Control Specialist	0x7B5AB0	0x7B40	Zainal T Arifin	Document Control Manager
219	Sean N Chai	Document Control Assistant	0x7B5AD0	0x7B40	Zainal T Arifin	Document Control Manager
220	Karen R Berge	Document Control Assistant	0x7B5AF0	0x7B40	Zainal T Arifin	Document Control Manager
221	Chris K Norred	Control Specialist	0x7B5B08	0x7B40	Zainal T Arifin	Document Control Manager
222	A. Scott  Wright	Master Scheduler	0x7BC0	0x	James R Hamilton	Vice President of Production
223	Sairaj L Uddin	Scheduling Assistant	0x7BD6	0x78	A. Scott  Wright	Master Scheduler
224	William S Vong	Scheduling Assistant	0x7BDA	0x78	A. Scott  Wright	Master Scheduler
225	Alan J Brewer	Scheduling Assistant	0x7BDE	0x78	A. Scott  Wright	Master Scheduler
226	Brian P LaMee	Scheduling Assistant	0x7BE1	0x78	A. Scott  Wright	Master Scheduler
227	Gary E. Altman	Facilities Manager	0x7C20	0x	James R Hamilton	Vice President of Production
228	Christian E Kleinerman	Maintenance Supervisor	0x7C2B	0x78	Gary E. Altman	Facilities Manager
229	Lori K Penor	Janitor	0x7C2B58	0x7C20	Christian E Kleinerman	Maintenance Supervisor
230	Stuart J Macrae	Janitor	0x7C2B68	0x7C20	Christian E Kleinerman	Maintenance Supervisor
231	Jo L Berry	Janitor	0x7C2B78	0x7C20	Christian E Kleinerman	Maintenance Supervisor
232	Pat H Coleman	Janitor	0x7C2B84	0x7C20	Christian E Kleinerman	Maintenance Supervisor
233	Magnus E Hedlund	Facilities Administrative Assistant	0x7C2D	0x78	Gary E. Altman	Facilities Manager
235	Paula M Barreto de Mattos	Human Resources Manager	0x8560	0x	Laura F Norman	Chief Financial Officer
236	Grant N Culbertson	Human Resources Administrative Assistant	0x856B	0x84	Paula M Barreto de Mattos	Human Resources Manager
237	Hao O Chen	Human Resources Administrative Assistant	0x856D	0x84	Paula M Barreto de Mattos	Human Resources Manager
238	Vidur X Luthra	Recruiter	0x856F	0x84	Paula M Barreto de Mattos	Human Resources Manager
239	Mindy C Martin	Benefits Specialist	0x857080	0x84	Paula M Barreto de Mattos	Human Resources Manager
240	Willis T Johnson	Recruiter	0x857180	0x84	Paula M Barreto de Mattos	Human Resources Manager
241	David J Liu	Accounts Manager	0x85A0	0x	Laura F Norman	Chief Financial Officer
242	Deborah E Poe	Accounts Receivable Specialist	0x85AB	0x84	David J Liu	Accounts Manager
243	Candy L Spoon	Accounts Receivable Specialist	0x85AD	0x84	David J Liu	Accounts Manager
244	Bryan A Walton	Accounts Receivable Specialist	0x85AF	0x84	David J Liu	Accounts Manager
245	Barbara C Moreland	Accountant	0x85B080	0x84	David J Liu	Accounts Manager
246	Dragan K Tomic	Accounts Payable Specialist	0x85B180	0x84	David J Liu	Accounts Manager
247	Janet L Sheperdigian	Accounts Payable Specialist	0x85B280	0x84	David J Liu	Accounts Manager
248	Mike K Seamans	Accountant	0x85B380	0x84	David J Liu	Accounts Manager
249	Wendy Beth Kahn	Finance Manager	0x85E0	0x	Laura F Norman	Chief Financial Officer
250	Sheela H Word	Purchasing Manager	0x85EB	0x84	Wendy Beth Kahn	Finance Manager
251	Mikael Q Sandberg	Buyer	0x85EB58	0x85E0	Sheela H Word	Purchasing Manager
252	Arvind B Rao	Buyer	0x85EB68	0x85E0	Sheela H Word	Purchasing Manager
253	Linda P Meisner	Buyer	0x85EB78	0x85E0	Sheela H Word	Purchasing Manager
254	Fukiko J Ogisu	Buyer	0x85EB84	0x85E0	Sheela H Word	Purchasing Manager
255	Gordon L Hee	Buyer	0x85EB8C	0x85E0	Sheela H Word	Purchasing Manager
256	Frank S Pellow	Buyer	0x85EB94	0x85E0	Sheela H Word	Purchasing Manager
257	Eric S Kurjan	Buyer	0x85EB9C	0x85E0	Sheela H Word	Purchasing Manager
258	Erin M Hagens	Buyer	0x85EBA2	0x85E0	Sheela H Word	Purchasing Manager
259	Ben T Miller	Buyer	0x85EBA6	0x85E0	Sheela H Word	Purchasing Manager
260	Annette L Hill	Purchasing Assistant	0x85EBAA	0x85E0	Sheela H Word	Purchasing Manager
261	Reinout N Hillmann	Purchasing Assistant	0x85EBAE	0x85E0	Sheela H Word	Purchasing Manager
262	David M Barber	Assistant to the Chief Financial Officer	0x8610	0x	Laura F Norman	Chief Financial Officer
264	Stephanie A Conroy	Network Manager	0x8D60	0x	Jean E Trenary	Information Services Manager
265	Ashvini R Sharma	Network Administrator	0x8D6B	0x8C	Stephanie A Conroy	Network Manager
266	Peter I Connelly	Network Administrator	0x8D6D	0x8C	Stephanie A Conroy	Network Manager
267	Karen A Berg	Application Specialist	0x8DA0	0x	Jean E Trenary	Information Services Manager
268	Ramesh V Meyyappan	Application Specialist	0x8DE0	0x	Jean E Trenary	Information Services Manager
269	Dan K Bacon	Application Specialist	0x8E10	0x	Jean E Trenary	Information Services Manager
270	François P Ajenstat	Database Administrator	0x8E30	0x	Jean E Trenary	Information Services Manager
271	Dan B Wilson	Database Administrator	0x8E50	0x	Jean E Trenary	Information Services Manager
272	Janaina Barreiro Gambaro Bueno	Application Specialist	0x8E70	0x	Jean E Trenary	Information Services Manager
274	Stephen Y Jiang	North American Sales Manager	0x9560	0x	Brian S Welcker	Vice President of Sales
275	Michael G Blythe	Sales Representative	0x956B	0x94	Stephen Y Jiang	North American Sales Manager
276	Linda C Mitchell	Sales Representative	0x956D	0x94	Stephen Y Jiang	North American Sales Manager
277	Jillian  Carson	Sales Representative	0x956F	0x94	Stephen Y Jiang	North American Sales Manager
278	Garrett R Vargas	Sales Representative	0x957080	0x94	Stephen Y Jiang	North American Sales Manager
279	Tsvi Michael Reiter	Sales Representative	0x957180	0x94	Stephen Y Jiang	North American Sales Manager
280	Pamela O Ansman-Wolfe	Sales Representative	0x957280	0x94	Stephen Y Jiang	North American Sales Manager
281	Shu K Ito	Sales Representative	0x957380	0x94	Stephen Y Jiang	North American Sales Manager
282	José Edvaldo Saraiva	Sales Representative	0x957440	0x94	Stephen Y Jiang	North American Sales Manager
283	David R Campbell	Sales Representative	0x9574C0	0x94	Stephen Y Jiang	North American Sales Manager
284	Tete A Mensa-Annan	Sales Representative	0x957540	0x94	Stephen Y Jiang	North American Sales Manager
285	Syed E Abbas	Pacific Sales Manager	0x95A0	0x	Brian S Welcker	Vice President of Sales
286	Lynn N Tsoflias	Sales Representative	0x95AB	0x94	Syed E Abbas	Pacific Sales Manager
287	Amy E Alberts	European Sales Manager	0x95E0	0x	Brian S Welcker	Vice President of Sales
288	Rachel B Valdez	Sales Representative	0x95EB	0x94	Amy E Alberts	European Sales Manager
289	Jae B Pak	Sales Representative	0x95ED	0x94	Amy E Alberts	European Sales Manager
290	Ranjit R Varkey Chudukatil	Sales Representative	0x95EF	0x94	Amy E Alberts	European Sales Manager
(end of results)
(290 rows affected)


	Solution Explained:
	-------------------
1) This solution uses an UNION ALL to put together the (1) list of employees at a management level directly under the CEO, (2) the CEO, and (3) the rest of the employees from lower hierarchical levels in the company.

2) The WHERE clause filters out all the employees and only retrieves the list of employees at upper management level, directly under the CEO, along with the CEO.

3) The second query is solely used to populate the column values "ManagerName" and "ManagerTitle" for the employees on the first query.

4) The second ON clause on the third query removes the rows belonging to the upper management employees and CEO, since the CEO's "OwnNode" is NULL.

5) The third query retrieves the rows belonging to the remaining employees (excluding upper management and the CEO). 


	Notes section:
	--------------
Below is the first query along with its output.
This query shows the list the list of employees at upper management level directly under the CEO, along with the CEO.

Column values for the CEO:
a) "OwnNode": this value is not available on this sample database. 
b) "ManagerNode", "ManagerName", "ManagerTitle": these are NULL values because the CEO does not have a manager.

Column values for upper management level directly under the CEO:
a) "OwnNode" (OrganizationNode): this value is different for each employee.
b) "ManagerNode": this value is the same for these employees, and equals to "0x".
c) "ManagerName" and "ManagerTitle": these are NULL values because the CEO's "OwnNode" value is not available on this sample database. 

The WHERE clause was used to retrieve these group of employees.

		Query #1:
		---------
USE AdventureWorks2022;
GO

SELECT	EmployeePerson.[BusinessEntityID] AS BusinessEntityID																-- Employee Details
	  , CONCAT(EmployeePerson.[FirstName], ' ', EmployeePerson.[MiddleName], ' ', EmployeePerson.[LastName]) AS EmployeeName
	  ,EmployeeTitle.[JobTitle] AS EmployeeTitle																		    -- Managers Details
	  ,EmployeeTitle.[OrganizationNode].GetAncestor(0) AS OwnNode
	  ,EmployeeTitle.[OrganizationNode].GetAncestor(1) AS ManagerNode
	  ,CAST(NULL AS NVARCHAR(150)) AS ManagerName
	  ,CAST(NULL AS NVARCHAR(50)) AS ManagerTitle
FROM [AdventureWorks2022].[Person].[Person] AS EmployeePerson
RIGHT JOIN [AdventureWorks2022].[HumanResources].[Employee] AS EmployeeTitle
ON EmployeePerson.[BusinessEntityID] = EmployeeTitle.[BusinessEntityID]
WHERE EmployeeTitle.[OrganizationNode].GetAncestor(1) = 0x OR EmployeeTitle.[OrganizationNode] IS NULL


		Output for Query #1:
		--------------------
BusinessEntityID	EmployeeName		EmployeeTitle			OwnNode		ManagerNode	ManagerName	ManagerTitle
1			Ken J Sánchez		Chief Executive Officer		NULL		NULL		NULL		NULL
2			Terri Lee Duffy		Vice President of Engineering	0x58		0x		NULL		NULL
16			David M Bradley		Marketing Manager		0x68		0x		NULL		NULL
25			James R Hamilton	Vice President of Production	0x78		0x		NULL		NULL
234			Laura F Norman		Chief Financial Officer		0x84		0x		NULL		NULL
263			Jean E Trenary		Information Services Manager	0x8C		0x		NULL		NULL
273			Brian S Welcker		Vice President of Sales		0x94		0x		NULL		NULL
