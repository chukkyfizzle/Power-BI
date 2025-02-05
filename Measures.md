Monthly_Hours_Bell = 
VAR CurrentDate = MAX('Date_Table'[Date]) // Current date in context

// Check if Project Name is in scope
VAR IsProjectNameInScope = ISINSCOPE(CRM_Review[Project Name])

// Get all relevant contracts for the current date
VAR RelevantContracts = 
    FILTER(
        CRM_Review,
        CRM_Review[Contract Start Date] <= CurrentDate &&
        DATEADD(CRM_Review[Contract Start Date], CRM_Review[Duration (Months)], MONTH) >= CurrentDate
    )

// Calculate the total hours for the current date
VAR Result = 
    SUMX(
        RelevantContracts,
        VAR StartDate = CRM_Review[Contract Start Date]
        VAR DurationMonth = CRM_Review[Duration (Months)]
        VAR Iteration = DATEDIFF(StartDate, CurrentDate, MONTH) + 1 // Months since start

        // Validate DurationMonth
        VAR ValidDurationMonth = 
            IF(DurationMonth > 0, DurationMonth, BLANK())

        // Calculate Month Mean
        VAR Month_Mean = 
            IF(
                ISEVEN(ValidDurationMonth),
                ValidDurationMonth / 2,
                (ValidDurationMonth - 1) / 2
            )

        // Calculate Month Divided by 4
        VAR Monthby4 = 
            DIVIDE(ValidDurationMonth, 4, 0)

        // Validate TotalHours and GoGetPercent
        VAR TotalHours = CRM_Review[Total Project Hours]
        VAR GoGetPercent = [GOGet%New]
        VAR Total_GoGet = TotalHours * GoGetPercent

        // Validate Iteration
        VAR ValidIteration = 
            IF(Iteration >= 1 && Iteration <= ValidDurationMonth, Iteration, BLANK())

        // Calculate Normal Distribution
        VAR Norm_Cal = 
            IF(
                NOT(ISBLANK(ValidIteration)) &&
                NOT(ISBLANK(Month_Mean)) &&
                NOT(ISBLANK(Monthby4)),
                NORM.DIST(ValidIteration - 1, Month_Mean, Monthby4, FALSE),
                BLANK()
            )

        // Calculate Sum of Previous Norm_Cal
        VAR Sum_Previous_Norm_Cal = 
            SUMX(
                FILTER(
                    ADDCOLUMNS(
                        GENERATESERIES(1, ValidIteration - 1, 1),
                        "Norm_Cal_Value", 
                        IF(
                            NOT(ISBLANK([Value])) &&
                            NOT(ISBLANK(Month_Mean)) &&
                            NOT(ISBLANK(Monthby4)),
                            NORM.DIST([Value] - 1, Month_Mean, Monthby4, FALSE),
                            BLANK()
                        )
                    ),
                    [Norm_Cal_Value] <> BLANK()
                ),
                [Norm_Cal_Value] * Total_GoGet
            )

        // Final Calculation
        RETURN
        IF(
            NOT(ISBLANK(ValidIteration)),
            IF(
                Iteration = DurationMonth,
                Total_GoGet - Sum_Previous_Norm_Cal,
                Norm_Cal * Total_GoGet
            ),
            BLANK()
        )
    )

// Handle aggregation at higher levels (e.g., Office level)
RETURN
IF(
    IsProjectNameInScope,
    Result, // Return the detailed calculation at Project Name level
    SUMX(VALUES(CRM_Review[Project Name]), Result) // Aggregate at higher levels
)
