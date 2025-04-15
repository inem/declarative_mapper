# DeclarativeMapper

`DeclarativeMapper` is a gem that allows you to declaratively define rules for transforming data from CSV files into Ruby objects using YAML configuration.

## Installation

```ruby
gem 'declarative_mapper'
```

```bash
bundle install
```

or

```bash
gem install declarative_mapper
```

## How It Works

DeclarativeMapper transforms data from CSV into Ruby hashes based on a YAML file with mapping rules and transformation methods.

1. You define the data structure in a YAML file
2. Create a module with methods for data transformation
3. The gem automatically applies these rules to CSV data

## Usage Example

### YAML file with mapping rules (customers.yml)

```yaml
mapping:
  name: (accountname,firstname,lastname)
  account_number: accountnum
  active: (status)
  billing_street: accountaddress
  billing_city: city
  billing_state: state
  billing_zip: postalcode
  billing_phone: phone_number_normalize(phonenumber)
  billing_country: country_us()
  address_attributes:
    street: accountaddress
    phone_attributes:
      code: (phonetype)
```

### Module with transformation methods

```ruby
module Reliable
  module MapperMethods
    module Customers
      def self.active(status)
        status == 'Active'
      end

      def self.name(*args)
        args.compact.map(&:strip).join(' ')
      end

      def self.country_us
        'United States'
      end

      def self.phone_number_normalize(number)
        "+#{number}"
      end
    end
  end
end
```

### Code usage

```ruby
require 'declarative_mapper'
require 'csv'
require 'yaml'

# Load mapping methods
Dir["reliable/**/*.rb"].each { |file| require_relative file }

# Read CSV
csv_path = "reliable/accounts.csv"
csv_table = CSV.parse(File.read(csv_path), headers: true)
csv_row = csv_table.first

# Read YAML with mapping rules
yml_path = "reliable/customers.yml"
yml_content = YAML.load_file(yml_path).deep_symbolize_keys

# Module with transformation methods
mapper_methods = Reliable::MapperMethods::Customers

# Transform data
customer_attrs = DeclarativeMapper.convert(mapper_methods, yml_content[:mapping], csv_row)

puts customer_attrs.inspect
```

## YAML File Syntax

- Simple copying: `field_name: csv_column_name`
- Method call: `field_name: (csv_column_name)`
- Method call with arguments: `field_name: method_name(csv_column1,csv_column2)`
- Nested attributes: use nested hashes in YAML
- Attribute arrays: use arrays in YAML

## License

MIT
