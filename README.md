Time
====
class MoviesController < ApplicationController

  def show
    id = params[:id] # retrieve movie ID from URI route
    @movie = Movie.find(id) # look up movie by unique ID
    # will render app/views/movies/show.<extension> by default
  end

  def index
    sort = params[:sort] || session[:sort]
    case sort
    when 'title'
      ordering,@title_header = {:order => :title}, 'hilite'
    when 'release_date'
      ordering,@date_header = {:order => :release_date}, 'hilite'
    end
    @all_ratings = Movie.all_ratings
    @selected_ratings = params[:ratings] || session[:ratings] || {}
    
    if @selected_ratings == {}
      @selected_ratings = Hash[@all_ratings.map {|rating| [rating, rating]}]
    end
    
    if params[:sort] != session[:sort] or params[:ratings] != session[:ratings]
      session[:sort] = sort
      session[:ratings] = @selected_ratings
      redirect_to :sort => sort, :ratings => @selected_ratings and return
    end
    @movies = Movie.find_all_by_rating(@selected_ratings.keys, ordering)
  end

  def new
    # default: render 'new' template
  end

  def create
    @movie = Movie.create!(params[:movie])
    flash[:notice] = "#{@movie.title} was successfully created."
    redirect_to movies_path
  end

  def edit
    @movie = Movie.find params[:id]
  end

  def update
    @movie = Movie.find params[:id]
    @movie.update_attributes!(params[:movie])
    flash[:notice] = "#{@movie.title} was successfully updated."
    redirect_to movie_path(@movie)
  end

  def destroy
    @movie = Movie.find(params[:id])
    @movie.destroy
    flash[:notice] = "Movie '#{@movie.title}' deleted."
    redirect_to movies_path
  end

end




#  This file is app/views/movies/index.html.haml
%h1 All Movies

= form_tag movies_path, :method => :get, :id => 'ratings_form' do
  = hidden_field_tag "title_sort", true if @title_header
  = hidden_field_tag ":release_date_sort", true if @date_header
  Include: 
  - @all_ratings.each do |rating|
    = rating
    = check_box_tag "ratings[#{rating}]", 1, @selected_ratings.include?(rating), :id => "ratings_#{rating}"
  = submit_tag 'Refresh', :id => 'ratings_submit'

%table#movies
  %thead
    %tr
      %th{:class => @title_header}= link_to 'Movie Title', movies_path(:sort => 'title', :ratings => @selected_ratings), :id => 'title_header'
      %th Rating
      %th{:class => @date_header}= link_to 'Release Date', movies_path(:sort => 'release_date', :ratings => @selected_ratings), :id => 'release_date_header'
      %th More Info
  %tbody
    - @movies.each do |movie|
      %tr
        %td= movie.title 
        %td= movie.rating
        %td= movie.release_date
        %td= link_to "More about #{movie.title}", movie_path(movie)

= link_to 'Add new movie', new_movie_path

class AddMoreMovies < ActiveRecord::Migration
  MORE_MOVIES = [
    {:title => 'Dabbang', :rating => 'PG', :release_date => '26-oct-2011'},
    {:title => 'Jab Rak Hai Jaan', :rating => 'PG', :release_date => '13-Nov-2012'},
    {:title => 'Student of The year', :rating => 'G', :release_date => '19-Oct-2012'},
    {:title => 'Time', :rating => 'PG-13', :release_date => '10-Sep-1992'},
    {:title => 'Chocolate', :rating => 'R', :release_date => '10-Jan-2001'},
    {:title => 'Three Idiots', :rating => 'PG-13', :release_date => '25-Apr-2008'},
    
  ]
  def up
    MORE_MOVIES.each do |movie|
      Movie.create!(movie)
    end
  end

  def down
    MORE_MOVIES.each do |movie|
      Movie.find_by_title_and_rating(movie[:title], movie[:rating]).destroy
    end
  end
end




# This file should contain all the record creation needed to seed the database with its default values.
# The data can then be loaded with the rake db:seed (or created alongside the db with db:setup).
#
# Examples:
#
#   cities = City.create([{ name: 'Chicago' }, { name: 'Copenhagen' }])
#   Mayor.create(name: 'Emanuel', city: cities.first)
Movies = [{:title => 'Dabbang', :rating => 'PG', :release_date => '26-oct-2011'},
    {:title => 'Jab Rak Hai Jaan', :rating => 'PG', :release_date => '13-Nov-2012'},
    {:title => 'Student of The year', :rating => 'G', :release_date => '19-Oct-2012'},
    {:title => 'Time', :rating => 'PG-13', :release_date => '10-Sep-1992'},
    {:title => 'Chocolate', :rating => 'R', :release_date => '10-Jan-2001'},
    {:title => 'Three Idiots', :rating => 'PG-13', :release_date => '25-Apr-2008'},
  
   ]

movies.each do |movie|
  Movie.create!(movie)
end

