flowchart TD

  %% ========== INTRO ==========
  Start[User starts topic]
  Intro1["SendActivity: Great!"]
  Intro2["SendActivity: I'm going to ask you some yes/no questions."]
  Intro3["SendActivity: Don't answer in chat, use buttons."]
  ReadyCard["Card: Ready to get started?"]
  ReadyYes{"readyToStart = 'Yes'?"}

  Start --> Intro1 --> Intro2 --> Intro3 --> ReadyCard --> ReadyYes

  %% ========== Q1: Developer agreement ==========
  Q1_Label["Q1: Developer agreement"]
  Q1_Card["Card: Is the land being sold<br/>in connection with a development agreement?"]
  Q1_Yes{"landDevelopmentAgreement = 'Yes'?"}

  ReadyYes -->|Yes| Q1_Label --> Q1_Card --> Q1_Yes

  Q1_YesYes["Answer: Subject to VAT at 13.5% due to development agreement.<br/>Vendor accounts for VAT, can recover costs, PCVEs, VAT clause etc."]
  Q1_YesEnd["BeginDialog: Questionhasbeenanswered"]

  Q1_Yes -->|Yes| Q1_YesYes --> Q1_YesEnd

  Q1_NoMsg["SendActivity: 👍 Land was not sold with a development agreement."]
  Q2_Label["Q2: Land partially developed?"]

  Q1_Yes -->|No| Q1_NoMsg --> Q2_Label

  %% ========== Q2: Partially developed? ==========
  Q2_Card["Card: Is the land partially developed?"]
  Q2_Yes{"landDeveloped = 'Yes'?"}

  Q2_Label --> Q2_Card --> Q2_Yes

  Q2_YesYesMsg["SendActivity: Land not sold with dev agreement AND partially developed."]
  Q2_YesAnswer["Answer: Partially developed land → VAT at 13.5%<br/>Vendor accounts, can recover costs, PCVEs, VAT clause etc."]
  Q2_End["BeginDialog: Questionhasbeenanswered"]

  Q2_Yes -->|Yes| Q2_YesYesMsg --> Q2_YesAnswer --> Q2_End

  %% ========== Q3: Developed to completion in last 5 yrs? ==========
  Q3_Label["Q3: Developed to completion in last 5 years?"]
  Q3_Card["Card: Has the land been developed to completion in the last 5 years?"]
  Q3_Yes{"landDevelopedToCompletionLast5Years = 'Yes'?"}

  Q2_Yes -->|No| Q3_Label --> Q3_Card --> Q3_Yes

  Q3_YesAnswer["Answer: Developed to completion in last 5 yrs → VAT at 13.5%<br/>Vendor accounts, can recover costs, PCVEs, VAT clause etc."]
  Q3_End["BeginDialog: Questionhasbeenanswered"]

  Q3_Yes -->|Yes| Q3_YesAnswer --> Q3_End

  %% ========== Q4: Confirm no development agreement ==========
  Q4_Label["Q4: Confirm no development agreement"]
  Q4_Card["Card: Confirm undeveloped land and no development agreement?"]
  Q4_Yes{"undevelopedLandNoDevAgreementConfirmed = 'Yes'?"}

  Q3_Yes -->|No| Q4_Label --> Q4_Card --> Q4_Yes

  %% ========== Q5: Joint Option to Tax (JOTT) ==========
  Q5_Label["Q5: Joint Option to Tax (JOTT)?"]
  Q5_Card["Card: Does the purchaser intend to exercise the JOTT?"]
  Q5_Yes{"jointOptionToTax = 'Yes'?"}

  Q4_Yes -->|Yes| Q5_Label --> Q5_Card --> Q5_Yes

  %% (If Q4 = No, user says there *is* a dev agreement – not shown in detail)
  Q4_No["(Q4 = No): There is a development agreement – branch not detailed here"]
  Q4_Yes -->|No| Q4_No

  %% ========== Q7: Purchaser suffers VAT cost under JOTT? ==========
  Q7_Label["Q7: Purchaser suffers VAT cost due to JOTT?"]
  Q7_Card["Card: Will the purchaser suffer a VAT cost as a result of the JOTT?"]
  Q7_Yes{"purchaserSuffersVATCostFromJOTT = 'Yes'?"}

  Q5_Yes -->|Yes| Q7_Label --> Q7_Card --> Q7_Yes

  Q7_YesAnswer["Answer: JOTT exercised and purchaser suffers VAT cost –<br/>detailed explanation on reverse charge, CGS, commercial implications, etc."]
  Q7_Yes_End1["BeginDialog: Questionhasbeenanswered"]
  Q7_Yes_End2["BeginDialog: thanksbye"]

  Q7_Yes -->|Yes| Q7_YesAnswer --> Q7_Yes_End1 --> Q7_Yes_End2

  %% If JOTT = No → go to Q8
  Q5_NoToQ8["(JOTT = 'No'): Proceed to clawback assessment"]
  Q5_Yes -->|No| Q5_NoToQ8

  %% ========== Q8: Clawback when JOTT not exercised ==========
  Q8_Label["Q8: Clawback if JOTT not exercised – did vendor recover VAT?"]
  Q8_Card["Card: Did the Vendor recover VAT on acquisition<br/>(or self‑account under JOTT)?"]
  Q8_Yes{"vendorRecoveredVATOnAcquisition = 'Yes'?"}

  Q5_NoToQ8 --> Q8_Label --> Q8_Card --> Q8_Yes

  Q8_YesAnswer["Answer: Vendor recovered VAT → clawback arises under CGS,<br/>consider JOTT, pricing, and impact on costs."]
  Q8_YesEnd["BeginDialog: Questionhasbeenanswered"]

  Q8_Yes -->|Yes| Q8_YesAnswer --> Q8_YesEnd

  Q8_NoAnswer["Answer: Vendor did not recover VAT → no clawback under CGS,<br/>sale exempt, no recovery of sales-related VAT."]
  Q8_NoEnd["BeginDialog: Questionhasbeenanswered"]

  Q8_Yes -->|No| Q8_NoAnswer --> Q8_NoEnd

  %% ========== Other branches not fully represented ==========
  Q5_No["(Q5 = No and other branches): goes through Q8 etc. (see YAML for full details)"]