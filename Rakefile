# encoding: UTF-8

require 'pry'
require 'json'
require 'yaml'
require 'nokogiri'
require 'active_support/all'

namespace :pc do
  desc "read data"
  task :read_data do
    data_tree = Nokogiri::HTML(File.read('data.html'))

    card_aggregations = {}
    general_aggregations = {}

    known_entities = begin
        YAML.load_file('known_entities.yml')
    rescue
        {}
    end

    index_to_col = [
        :details,
        :datum,
        :avisierungstext,
        :gutschrift,
        :lastschrift,
        :valuta,
        :saldo
    ]

    data_tree.css('table tbody tr').each do |tr|
        row = {}

        tr.css('td').each_with_index do |td, index|
            key = index_to_col[index]
            row[key] = Nokogiri::HTML(td.inner_html.gsub('<br>', "\n")).text.strip

            if [:gutschrift, :lastschrift].include? key
                row[key] = row[key].gsub('\'', '').to_f
            end
        end

        card_trans = row[:avisierungstext].match(/KARTEN NR\. (?<card>[0-9]+)/)

        if card_trans
            aggregation = card_aggregations[card_trans[:card]] ||= {
                lastschrift: 0,
                gutschrift: 0,
                vendors: []
            }

            aggregation[:lastschrift] += row[:lastschrift]
            aggregation[:gutschrift] += row[:gutschrift]

            text = row[:avisierungstext].match(/(.+\n){3}(?<vendor_name>.+)\n.*/)

            if text && text[:vendor_name]
                vendor_aggregation = aggregation[:vendors].select{|val| val[:name] == text[:vendor_name]}.first

                if vendor_aggregation.blank?
                    vendor_aggregation = {
                        name: text[:vendor_name],
                        transactions: []
                    }
                    aggregation[:vendors] << vendor_aggregation
                end

                vendor_aggregation[:transactions] << row
            end
        end

        [:gutschrift, :lastschrift].each do |key|
            value = row[key]
            return unless value.present?

            source = 'unknown'
            known_entities[key.to_s].to_a.each do |name, regex|
                if regex.match(row[:avisierungstext])
                    source = name
                    break
                end
            end

            general_aggregations[key] ||= {}
            aggregation = general_aggregations[key][source] ||= {
                key => 0,
                transactions: []
            }
            aggregation[key] += value
            aggregation[:transactions] << row
        end
    end

    card_aggregations.each do |key, aggregation|
        aggregation[:vendors].each do |vendor|
            vendor[:lastschrift] = vendor[:transactions].inject(0){|sum, transaction| sum + transaction[:lastschrift]}
            vendor[:gutschrift] = vendor[:transactions].inject(0){|sum, transaction| sum + transaction[:gutschrift]}
            vendor[:frequency] = vendor[:transactions].length
        end
        aggregation[:vendors] = aggregation[:vendors].sort_by{|v| v[:lastschrift]}.reverse
    end

    File.open('data.json', 'wb') do |file|
        file.write JSON.pretty_generate({
            cards: card_aggregations,
            general: general_aggregations
        })
    end
  end
end