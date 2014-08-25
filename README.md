
 def self.search(search)
      self.where('lower(name) like lower(?) or lower(description) like lower(?)', "%#{search}%", "%#{search}%").order("name ASC")
  end
  
  def self.versions(params)
    @drug_classes = DrugClass.includes( {:drugs => [:strengths, :drug_warning_precautions, :interactions]})

    @drugs_all =@drugs_updated = 0
    @drug_classes.each do |drug_class|
  
       drug_class.drugs.each do |drug|
        @drugs_all += 1
        @drugs_updated += 1 if drug.is_updated_to_version_two
      end
    end
            
    percentage = (@drugs_updated.to_f / @drugs_all.to_f) * 100
    @percent_complete = sprintf('%.2f', percentage)
    
    @drug_hash = {}
    @drug_classes.sort_by{|c| c.name}
     @drug_classes.each do | drug_class|
      
      @drugs = drug_class.drugs
      if params[:filter_by_converted].to_s == "true"
        @drugs.map{|x| !x.is_updated_to_version_two}
      elsif params[:filter_by_converted].to_s == "false"
        @drugs.map{|x| x.is_updated_to_version_two}
      end
      if !@drugs.nil?
        @drugs.sort_by(&:display_name)
        @drug_hash[drug_class.name] = @drugs
        @drug_hash_sorted = @drug_hash.sort
      end
    end
    @drug_hash_sorted.delete_if{|key, value| value.size < 1} 
    return @drug_hash_sorted,@drugs_updated,@drugs_all
  end
end
