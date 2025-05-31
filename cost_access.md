createOrReplace

	table Cost_User_Access
		lineageTag: 263bed44-e6d9-499c-a44e-eb5be4158da8

		measure check_user =
				
				var usename = USERNAME()
				
				var x = calculate(
				    count(Cost_User_Access[Cost_User_Access]),
				    filter(
				        Cost_User_Access,
				        Cost_User_Access[Cost_User_Access] = usename
				    )
				)
				return
				x
			formatString: 0
			lineageTag: 5431995c-d4b2-47dd-9cef-4e960050fd1a

		measure cost_url_selection =
				
				var costURL = "https://app.powerbi.com/groups/me/apps/37754877-d1b7-47d9-b5ef-a9e009662faa/reports/c370ffc7-0a3c-4d55-a139-f713a097009d/ReportSection4080c7c3dea3a7efaba6?ctid=0843acec-fd3e-49be-bd54-18c6048a3fd0&experience=power-bi"
				
				return
				if([check_user] = 1,costURL, BLANK())
			lineageTag: 91d18632-a60d-4673-92aa-d0010ffd3043

		column Cost_User_Access
			dataType: string
			lineageTag: 026177ff-3d1f-436c-9241-71ec5e905862
			summarizeBy: none
			sourceColumn: Cost_User_Access

			annotation SummarizationSetBy = Automatic

		partition Cost_User_Access-b2a75235-fa6d-441c-b6f9-37c7bd16f07d = m
			mode: import
			source =
					let
					    Source = Excel.Workbook(Web.Contents(Source_Cost_User_Access), null, true),
					    Sheet1_Sheet = Source{[Item="Sheet1",Kind="Sheet"]}[Data],
					    #"Changed Type" = Table.TransformColumnTypes(Sheet1_Sheet,{{"Column1", type text}}),
					    #"Promoted Headers" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"Cost_User_Access", type text}}),
					    #"Filtered Rows" = Table.SelectRows(#"Changed Type1", each ([Cost_User_Access] <> null))
					in
					    #"Filtered Rows"

		annotation PBI_ResultType = Table

