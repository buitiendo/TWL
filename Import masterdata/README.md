# Chủ đề: Base import masterdata
# Gem được sử dụng
* Activerecord-Import
# Tao Base
## 1. Model
* Tao module M quan ly cac model import
``` Ruby
app/models/m.rb 

module M
  def self.table_name_prefix
    "m_"
  end
end
```
* Tao model
``` Ruby
app/models/m/category.rb

class M::Category < ApplicationRecord
end
```

## 2. Service
* Import CSV service
``` Ruby
app/services/import_csv_service.rb

class ImportCsvService
  attr_reader :table_name, :csv_path, :validate

  def initialize table_name, csv_path, validate = true
    @table_name = table_name
    @csv_path = csv_path
    @validate = validate
  end

  def perform
    ActiveRecord::Base.transaction do
      model.delete_all
      model.import records, batch_size: 10_000, validate: validate
    end
  rescue StandardError => e
    Rails.logger.error e.message
  end

  private
  def model
    @model ||= parse_table_name
  end

  def records
    @records ||= CSV.foreach(csv_path, headers: true).map do |row|
      model.new row.to_hash
    end
  end

  def parse_table_name
    table_name.gsub(/\Am_/, "m::_").classify.constantize
  end
end
```
* Import master data service
``` Ruby
app/services/import_master_data_service.rb

class ImportMasterDataService
  MASTER_TABLES = %w[m_categories m_xyz].freeze

  def perform
    MASTER_TABLES.each do |table_name|
      csv_path = Rails.root.join "db", "seeds", "#{table_name}.csv"
      ImportCsvService.new(table_name, csv_path, false).perform
    end
  end
end
```
