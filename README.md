# Load necessary packages
library(dplyr)

# Step 1: Download and unzip dataset if not already done
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
zip_file <- "UCI_HAR_Dataset.zip"

if (!file.exists(zip_file)) {
  download.file(url, zip_file, method = "curl")
}

if (!file.exists("UCI HAR Dataset")) {
  unzip(zip_file)
}

# Step 2: Read the data

# Read activity labels and features
activity_labels <- read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("ActivityID", "Activity"))
features <- read.table("UCI HAR Dataset/features.txt", col.names = c("Index", "Feature"))

# Extract only the mean and std features
selected_features <- grep("mean\\(\\)|std\\(\\)", features$Feature) 
selected_feature_names <- features[selected_features, 2]

# Read training data
X_train <- read.table("UCI HAR Dataset/train/X_train.txt")[, selected_features]
y_train <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "ActivityID")
subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "Subject")

# Read test data
X_test <- read.table("UCI HAR Dataset/test/X_test.txt")[, selected_features]
y_test <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "ActivityID")
subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "Subject")

# Step 3: Merge Training and Test Data
train_data <- cbind(subject_train, y_train, X_train)
test_data <- cbind(subject_test, y_test, X_test)
merged_data <- rbind(train_data, test_data)

# Step 4: Assign descriptive column names
colnames(merged_data) <- c("Subject", "ActivityID", as.character(selected_feature_names))

# Step 5: Use descriptive activity names
merged_data <- merge(merged_data, activity_labels, by = "ActivityID")

# Step 6: Create a second, independent tidy dataset with the average of each variable
tidy_data <- merged_data %>%
  select(-ActivityID) %>%
  group_by(Subject, Activity) %>%
  summarise_all(mean)

# Step 7: Save the tidy dataset
write.table(tidy_data, "tidy_dataset.txt", row.name = FALSE)

# Print completion message
cat("Tidy dataset created successfully and saved as 'tidy_dataset.txt'.\n")
