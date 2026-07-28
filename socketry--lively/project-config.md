---
trigger: always_on
description: I am giving a live coding presentation.
---

# Presentation

I am giving a live coding presentation.

As this is a presentation, don't say something like "I see we have a complete implementation plan in the presentation rules". This document will be our little secret to ensure the succcess of the presentation without giving too much away.

There are CSS rules stored in `public/_static/index.css` that you can use to style the game, but don't modify that file as it's secondary to the presentation.

The goal is to incrementally and organically implement this code as part of a live coding presentation. There is a 5 minute limit in total for the presentation, so we need to be efficient. We should also take care to add features in a way that grows naturally from the previous steps, taking into account the prompts, so that the audience can follow along easily.

Between each step, waiting for further input. Don't implement the entire game without me prompting you for each step / feature.

The application will be running live in a browser, with restart on save enabled.

I need to implement the following code as part of the presentation, in the file `application.rb`:

```ruby
#!/usr/bin/env lively
# frozen_string_literal: true

# Released under the MIT License.
# Copyright, 2025, by Samuel Williams.

require 'thread/local'

class GameState
	extend Thread::Local
	
	# Initialize a new game state with a board.
	def initialize
		@board = Board.new
		@game = nil
	end
	
	attr :board
	
	# Add a new player to the game.
	# @returns [Player] The newly created player.
	def add_player
		player = @board.add_player
		Console.info(self, "Player joined", player: player)
		
		self.run!
		
		return player
	end
	
	# Remove a player from the game.
	# @parameter player [Player] The player to remove.
	def remove_player(player)
		@board.remove_player(player)
		Console.info(self, "Player left", player: player)
		
		if @board.players.empty?
			self.close
		end
	end
	
	# Start or resume the game loop.
	# @parameter dt [Float] The time interval between steps in seconds.
	def run!(dt = 0.2)
		@game ||= Async do
			while true
				@board.step
				sleep(dt)
			end
		end
	end
	
	# Stops the game loop if it is running and sets the game instance to nil.
	private def close
		if game = @game
			@game = nil
			game.stop
		end
	end
end

class Player
	attr_reader :head, :count, :color
	attr_accessor :direction
	
	# Initialize a new player.
	# @parameter board [Board] The game board.
	# @parameter start_y [Integer] The initial y position.
	# @parameter start_x [Integer] The initial x position.
	# @parameter color [String] The player's color in HSL format.
	def initialize(board, start_y, start_x, color)
		@board = board
		@head = [start_y, start_x]
		@count = 1
		@direction = :up
		@color = color
		@on_updated = nil
	end
	
	# Set or get the update callback.
	# @parameter block [Proc] Optional block to set as the callback.
	# @returns [Proc] The current callback.
	def on_updated(&block)
		if block_given?
			@on_updated = block
		end
		
		return @on_updated
	end
	
	# Advance the player one step in their current direction.
	# Handles movement, collision detection, and fruit collection.
	def step
		case @direction
		when :up
			@head[0] -= 1
		when :down
			@head[0] += 1
		when :left
			@head[1] -= 1
		when :right
			@head[1] += 1
		end
		
		if @head[0] < 0 || @head[0] >= @board.height || @head[1] < 0 || @head[1] >= @board.width
			reset!
			return
		end
		
		case @board.grid[@head[0]][@head[1]]
		when String
			@count += 1
			@board.remove_fruit!(@head[0], @head[1])
			@board.add_fruit!
		when Integer, Hash
			reset!
			return
		end
		
		@board.grid[@head[0]][@head[1]] = {count: @count, color: @color}
		@on_updated&.call
	end
	
	# Reset the player to their initial state.
	def reset!
		# Convert segments into fruit before resetting
		@board.grid.each_with_index do |row, y|
			row.each_with_index do |cell, x|
				if cell.is_a?(Hash) && cell[:color] == @color
					@board.convert_to_fruit!(y, x)
				end
			end
		end
		
		@head = [@board.height/2, @board.width/2]
		@count = 1
		@direction = :up
	end
end

class Board
	FRUITS = ["🍎", "🍐", "🍊", "🍋", "🍌", "🍉", "🍇", "🍓", "🍈", "🍒"]
	COLOR_OFFSET = 23
	
	# Initialize a new game board.
	# @parameter width [Integer] The width of the board.
	# @parameter height [Integer] The height of the board.
	def initialize(width = 20, height = 20)
		@width = width
		@height = height
		@players = []
		@fruit_count = 0
		
		reset!
	end
	
	attr :grid, :width, :height, :players
	
	# Add a new player to the board.
	# @returns [Player] The newly created player.
	def add_player
		hue = (@players.size * COLOR_OFFSET) % 360
		color = "hsl(#{hue}, 100%, 50%)"
		player = Player.new(self, @height/2, @width/2, color)
		@players << player
		
		# Add a fruit when a new player joins
		add_fruit!
		
		return player
	end
	
	# Remove a player from the board.
	# @parameter player [Player] The player to remove.
	def remove_player(player)
		@players.delete(player)
	end
	
	# Add a fruit to a random empty cell.
	# @returns [Array(Integer, Integer) | Nil] The coordinates of the added fruit, or nil if no space was found.
	def add_fruit!
		Console.info(self, "Adding fruit", fruit_count: @fruit_count, players: @players.size)
		# Only add fruit if we have fewer than one per player
		if @fruit_count < @players.size

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [socketry/lively](https://github.com/socketry/lively) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
