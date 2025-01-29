% Simulating Data Extraction from SQLite Database
conn = sqlite('sample_database.db'); % Connect to database
data = fetch(conn, 'SELECT * FROM sample_table'); % Fetch data
close(conn); % Close connection

% Convert data to table for easy processing
dataTable = cell2table(data, 'VariableNames', {'ID', 'Age', 'Salary', 'Gender', 'Country'});

% Display original data
disp('Original Data:');
disp(dataTable);

%% Data Pre-processing

% 1. Handling Missing Values (Filling NaN with Mean for Numerical Columns)
numericCols = {'Age', 'Salary'};
for i = 1:length(numericCols)
    col = numericCols{i};
    dataTable.(col)(isnan(dataTable.(col))) = mean(dataTable.(col), 'omitnan');
end

% 2. Removing Duplicates
dataTable = unique(dataTable, 'rows');

% 3. Encoding Categorical Variables (One-Hot Encoding for 'Gender' and 'Country')
dataTable.Gender = categorical(dataTable.Gender);
dataTable.Country = categorical(dataTable.Country);

% Convert categorical columns into numerical representation
dataTable = [dataTable(:, {'ID', 'Age', 'Salary'}), ...
             table(double(dataTable.Gender), double(dataTable.Country), ...
                   'VariableNames', {'Gender_Encoded', 'Country_Encoded'})];

% 4. Normalization (Min-Max Scaling for Age and Salary)
colsToNormalize = {'Age', 'Salary'};
for i = 1:length(colsToNormalize)
    col = colsToNormalize{i};
    dataTable.(col) = (dataTable.(col) - min(dataTable.(col))) / ...
                      (max(dataTable.(col)) - min(dataTable.(col)));
end

% Display pre-processed data
disp('Pre-Processed Data:');
disp(dataTable);
