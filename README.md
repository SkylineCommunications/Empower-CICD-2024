# .NET Tools

[DataMiner Docs](https://docs.dataminer.services/develop/CICD/Platform_Independent_CICD.html)

## Validator

```yml
    # Install the Validator .NET tool so we can check the connector solution for mistakes.
    - name: Install Connector Validator
      run: dotnet tool install -g Skyline.DataMiner.CICD.Tools.Validator

    # Run the Validator .NET tool. We give the workspace as an input so that the tool can find the solution file. The output will be the results in the form of JSON and HTML.
    - name: Run Connector Validator
      run: dataminer-validator validate-protocol-solution --solution-path "${{ github.workspace }}" --output-directory "${{ github.workspace }}" --output-file-name "validateResults" 

    # Archive the results so that they are accesible via the UI.
    - name: Archive Results
      if: success()
      uses: actions/upload-artifact@v4
      with:
        name: validateResults
        path: ${{ github.workspace }}/validateResults.*

    # Read out the validator results and define the quality thresholds. Based on the thresholds, this step will succeed or fail.
    - name: Quality Gate
      run: |
        json=$(cat "${{ github.workspace }}/validateResults.json")
        critical=$(echo "$json" | jq -r '.CriticalIssueCount')
        major=$(echo "$json" | jq -r '.MajorIssueCount')

        if [ "$critical" != "0" ] || [ "$major" != "0" ]; then
          echo "Error: CriticalIssueCount or MajorIssueCount is not 0"
          exit 1
        fi

```

## Validator Fix

```xml
		<Param id="1" trending="false">
			<Name>MyString</Name>
			<Description>My String</Description>
			<Type>read</Type>
			<Information>
				<Subtext>My String</Subtext>
			</Information>
			<Interprete>
				<RawType>other</RawType>
				<Type>string</Type>
				<LengthType>next param</LengthType>
			</Interprete>
			<Display>
				<RTDisplay>true</RTDisplay>
				<Positions>
					<Position>
						<Page>General</Page>
						<Column>0</Column>
						<Row>0</Row>
					</Position>
				</Positions>
			</Display>
			<Measurement>
				<Type>string</Type>
			</Measurement>
		</Param>
```

## Package Creation

> [!NOTE]
> In the workflow, by default, the first job is called `build`. Change this to `CI`.

```yml
  CD:
    runs-on: windows-latest
    # CI is the name of the previous job. If the CI job fails, then this job will not be executed.
    needs: CI
    steps:
    - uses: actions/checkout@v4
  
    # Install the Packager .NET tool so we can create a dmprotocol package from the connector solution.
    - name: Install Package Creation
      run: dotnet tool install -g Skyline.DataMiner.CICD.Tools.Packager

    # Run the Packager .NET tool to create a dmprotocol package.
    - name: Create Protocol Package
      run: dataminer-package-create dmprotocol "${{ github.workspace }}" --name "protocol" --output "${{ github.workspace }}\\_PackageResults"

    # Archive the dmprotocol package so you can access it via the UI.
    - name: Archive Package
      if: success()
      uses: actions/upload-artifact@v4
      with:
        name: packageResults
        path: ${{ github.workspace }}\\_PackageResults\\*

```

## See also

[Kata #22: How to make a connector CI/CD pipeline](https://docs.dataminer.services/develop/CICD/Tutorials/CICD_Tutorial_Connector.html)
