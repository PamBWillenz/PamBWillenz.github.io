---
layout: post
title: Up All Knight
feature-img: "img/sample_feature_img_2.png"

thumbnail-path: "img/chess_thumbnail.png"

short-description: Chess game with all the trimmings!

---
[Up All Knight](https://chess-up-all-knight.herokuapp.com/) is a chess app built on Ruby on Rails, JavaScript, jQuery and has special features: blitz chess (chess with a timer) or standard (no timer), captured pieces and rankings. Includes TDD/RSpec testing with 83% coverage.

<img src="/img/chess_landing_page.png">
<br>

<h3> Player's Create a Game </h3>

<img src="/img/chess_create_game.png">
<br>

<h3> Chess Landing Page </h3>

<img src="/img/chess_landing_page.png">
<br>

<h3> Chessboard Game </h3>

<img src="/img/chess_chessboard_game_show.png">
<br>

<h3> Explanation and Challenges </h3>

1. Work on Agile/Scrum team with six developers to build a working chess app with features: 
  + chessboard visualizations, 
  + color choices (black or white), 
  + chess pieces moves, 
  + timers, 
  + captured pieces,
  + player readiness, draw, forfeit 
  + rankings

2. Understand Logic behind Chess piece moves:
  + How would you validate that a queen can move to a particular location on the board, given where the other pieces are on the board?

<h3> Results </h3>

  {% highlight ruby%}
class Queen < Piece

  def valid_move?(destination_x, destination_y)
    valid = super(destination_x, destination_y)
    valid && (
      horizontal(destination_y) || 
      vertical(destination_x) || 
      diagonal(destination_x, destination_y)
      )
  end
  
  def unicode_symbol
    if self.get_color == WHITE
      return "&#9813;"
    else
      return "&#9819;"
    end
  end

end
{% endhighlight %}

  + How would you enable the castling move?

{% highlight ruby %}
class King < Piece

    
  def valid_move?(destination_x, destination_y)
    valid = super(destination_x, destination_y)
    if valid
      if can_castle?(destination_x, destination_y)
        return true
      elsif (destination_x - self.x_position).abs > 1 || (destination_y - self.y_position).abs > 1
        valid = false
      end
    end
    return valid
  end


  def move_to(destination_x, destination_y)
    valid = super
    if valid
      castle!(destination_x, destination_y) 
    end
    valid
  end

  def can_castle?(destination_x, destination_y)
    return self.moves == 0 && (castling_kingside?(destination_x, destination_y) || 
    castling_queenside?(destination_x, destination_y)) && (game.check?(player)) == false
  end

  def castle!(destination_x, destination_y)
    if castling_kingside?(destination_x, destination_y)
      rook = game.pieces.where(x_position: 7, y_position: destination_y, player_id: player_id, type: "Rook").last
      rook.update_attributes(x_position: 5)
    end
    if castling_queenside?(destination_x, destination_y)
      rook = game.pieces.where(x_position: 0, y_position: destination_y, player_id: player_id, type: "Rook").last 
      rook.update_attributes(x_position: 3)
    end
  end

  def castling_kingside?(destination_x, destination_y)
    return (destination_x == 6 && destination_y == y_position) && 
        (is_rook_kingside?(destination_y))
  end

  def castling_queenside?(destination_x, destination_y)
    return (destination_x == 2 && destination_y == y_position) && 
        (is_rook_queenside?(destination_y))
  end

  def is_rook_queenside?(destination_y)
    rook = game.pieces.where(x_position: 0, y_position: destination_y, player_id: player_id, type: "Rook").last
    if rook != nil
      if rook.moves == 0 
        return true
      end
    end
    return false
  end

  def is_rook_kingside?(destination_y)
    rook = game.pieces.where(x_position: 7, y_position: destination_y, player_id: player_id, type: "Rook").last 
    if rook != nil
      if rook.moves == 0 
        return true
      end
    end
    return false
  end


  def unicode_symbol
    if self.get_color == WHITE
      return "&#9812;"
    else
      return "&#9818;"
    end
  end
end

{% endhighlight %}  


  + How would you update the board for user #2, when user #1 moves the piece without requiring a page refresh. We used pusher, a websocket applications with social features that enables realtime user activity. 
{% highlight ruby %}
class PusherController < ApplicationController
  skip_before_filter :verify_authenticity_token
  
  def auth
    
    if current_player && params[:channel_name] == "private-user_#{current_player.id}"
      render :json => Pusher[params[:channel_name]].authenticate(params[:socket_id])
    else
      render :text => "Not authorized", :status => '403'
    end
  end
end

def player_ready
    if @game.is_blitz
      if !@game.white_ready && !@game.black_ready
        Pusher["private-user_#{@opponent_id}"].trigger!('start_ready_timer', {
          :message => "Start Ready Timer."
          })
      end
      if params[:current_player].to_i == @game.white_player_id
        Pusher["private-user_#{@opponent_id}"].trigger!( "message", {
          :message => "White Player is ready for blitz chess game."
          })
          @game.white_ready = true
          @game.save
      elsif params[:current_player].to_i == @game.black_player_id
        Pusher["private-user_#{@opponent_id}"].trigger!( "message", {
          :message => "Black Player is ready for blitz chess game."
          })
          @game.black_ready = true
          @game.save
      end
      if @game.white_ready && @game.black_ready
        @game.has_started = true
        @game.save
        Pusher["broadcast_#{@game.id}"].trigger!('start_game', {
          has_started: true })
        Pusher["broadcast_#{@game.id}"].trigger!('clear_ready_timer', {
          })
        Pusher["broadcast_#{@game.id}"].trigger!('hide_ready_buttons', {})
      end
    end
    render json: {}
  end
{% endhighlight %}

  + How would you check for checkmate/check?
{% highlight ruby %}
def check?(player)
    check = false
    king = pieces.where(type: 'King', player: player).last
    opponents_pieces = pieces.where(player: opponent_player(player), captured: false)
    if king !=nil
      return opponents_pieces.any? { |piece| piece.valid_move?(king.x_position, king.y_position) }
    end
      return false
  end

  def checkmate?(player)
    if check?(player)
      checkmate = true
      king = pieces.where(type: 'King', player: player).last
      (0..7).each do |x|
        (0..7).each do |y|
          if king.valid_move?(x, y)
            original_x = king.x_position
            original_y = king.y_position
            captured_piece = pieces.where(x_position: x,y_position: y, game_id: id).last
            king.move_to(x, y)
            checkmate = false if !check?(player)
            king.move_to(original_x, original_y)
            if captured_piece != nil
              captured_piece.reload.update_attributes(x_position: x, y_position: y, captured: false)
            end
          end
        end
      end
      return checkmate
    end
    return false
  end
{% endhighlight %}

<h3> Test using RSpec </h3>

{% highlight ruby %}
require 'rails_helper'

RSpec.describe Game, type: :model do

  let(:white_player) { FactoryGirl.create(:player, email: 'blah@blah.com', password: 'SPACECAT') }
  let(:black_player) { FactoryGirl.create(:player, email: 'meow@meow.com', password: 'MONORAILCAT') }
  let(:game) { FactoryGirl.create(:game, :populated, white_player_id: white_player.id, black_player_id: black_player.id) }
  let(:king) { game.pieces.where(:player_id => black_player.id, :type => "King").last }
  let(:rook) { FactoryGirl.create(:rook, player_id: white_player.id) }

  describe "check?" do
    let(:game) { FactoryGirl.create(:game, white_player: white_player, black_player: black_player) }

    context "the black king" do
      it "should be in check" do
        king = FactoryGirl.create(:king, x_position: 4, y_position: 1, player: black_player, game: game)
        rook = FactoryGirl.create(:rook, x_position: 4, y_position: 7, player: white_player, game: game)
        expect(game.check?(black_player)).to eq (true)
      end

      it "should not be in check" do
        king = FactoryGirl.create(:king, x_position: 4, y_position: 1, player: black_player, game: game)
        rook = FactoryGirl.create(:rook, x_position: 6, y_position: 7, player: white_player, game: game)
        expect(game.check?(black_player)).to eq (false)
      end
    end

    context "the white king" do
      it "should be in check" do
        king = FactoryGirl.create(:king, x_position: 4, y_position: 1, player: white_player, game: game)
        rook = FactoryGirl.create(:rook, x_position: 4, y_position: 7, player: black_player, game: game)
        expect(game.check?(white_player)).to eq (true)
      end

      it "should not be in check" do
        king = FactoryGirl.create(:king, x_position: 4, y_position: 1, player: white_player, game: game)
        rook = FactoryGirl.create(:rook, x_position: 6, y_position: 7, player: black_player, game: game)
        expect(game.check?(white_player)).to eq (false)
      end
    end
  end

  describe "checkmate?" do
    let(:game) { FactoryGirl.create(:game, white_player_id: white_player.id, black_player_id: black_player.id) }

    it "should be in checkmate if king is currently in check and cannot move out of check" do
      king = FactoryGirl.create(:king, x_position: 0, y_position: 0, player: black_player, game: game)
      queen = FactoryGirl.create(:queen, x_position: 0, y_position: 1, player: white_player, game: game)
      queen_2 = FactoryGirl.create(:queen, x_position: 1, y_position: 1, player: white_player, game: game)
      expect(game.reload.checkmate?(black_player)).to eq(true)
    end

    it "should have checkmate value equal to false if king is not in check" do
      game.check?(black_player) == false
      expect(game.checkmate?(black_player)).to eq(false)
    end
  end
{% endhighlight %}

<h3> Conclusion </h3>

On the team, we wanted the chess app to have solid logic to move the pieces as conventional chess pieces and also have the game be asychronous, using JavaScript/jQuery. We also wanted the front end to be clean, inviting and easy to navigate. 
